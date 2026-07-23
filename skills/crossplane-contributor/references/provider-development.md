# Provider development, in depth

Source: [crossplane provider development guide][provider-dev]. This covers
what makes a Crossplane infrastructure resource and how to add a new one
to a provider.

## The shape of a managed service

Typical Crossplane infrastructure consists of two API kinds and one
controller:

1. **A managed resource** — a cluster-scoped, high-fidelity representation
   of a resource in an external system (e.g. a cloud API). Managed
   resources are *not* portable across external systems; they're tightly
   coupled to the implementation details of the thing they represent.
2. **A `ProviderConfig`** — indicates how to authenticate to the external
   system, typically via a referenced Secret, plus any other connection
   metadata.

A controller reconciles the managed resource with the external system: it
observes desired state (the resource spec), observes actual state (calling
the external API), and drives actual state toward desired state.

## Starting point

- **New provider from scratch**: use the [provider-template] repo ("Use
  this template" in GitHub), not bare kubebuilder — it already encodes
  Crossplane's conventions, which have diverged from vanilla kubebuilder
  scaffolds. For Terraform-backed providers, use [Upjet] instead.
- **New resource in an existing provider**: it's often faster and safer to
  copy and adapt a sibling resource already in the repo than to regenerate
  from kubebuilder and re-apply all the Crossplane-specific adjustments by
  hand.
- Kubebuilder scaffold command, if you do start there:
  ```sh
  kubebuilder create api \
      --group example --version v1alpha1 --kind FavouriteDBInstance \
      --resource=true --controller=false --namespaced=false
  ```

## Managed resource requirements

A managed resource type **must**:

- Satisfy crossplane-runtime's `resource.Managed` interface.
- Embed `xpv1.ResourceStatus` in its `Status` struct.
- Embed `xpv1.ResourceSpec` **and** a `Parameters` struct in its `Spec`
  struct.
- Carry `+kubebuilder:subresource:status` and
  `+kubebuilder:resource:scope=Cluster` comment markers.

`Parameters` should be a **high-fidelity** mirror of the external API's
writeable fields — as close to the provider's own shape as Kubernetes API
conventions allow. Only reshape where conventions demand it:

```go
// External API: {"fanciness_level": 100} → Kubernetes: fancinessLevel

// FavouriteDBInstanceParameters define the desired state of an instance.
// Most fields map directly to https://favourite.example.org/api/v1/db#Instance
type FavouriteDBInstanceParameters struct {
	// Name of this instance.
	Name string `json:"name"`

	// FancinessLevel specifies exactly how fancy this instance is.
	FancinessLevel int `json:"fancinessLevel"`

	// Version specifies what version of FancySQL this instance will run.
	// +optional
	Version *string `json:"version,omitempty"`
}
```

Output-only fields (IDs, computed status, hostnames) belong in `Status`,
never in `Parameters`:

```go
type FavouriteDBInstanceStatus struct {
	xpv1.ResourceStatus `json:",inline"`

	ID       int    `json:"id,omitempty"`
	Status   string `json:"status,omitempty"`
	Hostname string `json:"hostname,omitempty"`
}
```

Document every field with GoDoc written for someone running
`kubectl explain` or reading generated API reference docs — assume they are
not reading the Go source.

## ProviderConfig requirements

A `ProviderConfig` type **must**:

- Be named exactly `ProviderConfig`.
- Embed `xpv1.ProviderSpec` in its `Spec` struct.
- Carry `+kubebuilder:resource:scope=Cluster`.

```go
type ProviderSpec struct {
	xpv1.ProviderSpec `json:",inline"`

	// Any auth info not already covered by the embedded Secret reference.
	ProjectID string `json:"projectID"`
}
```

## Finishing the API types

- Add any kubebuilder comment markers useful for validation or extra
  `kubectl get` columns.
- Run `make reviewable` to regenerate CRDs and crossplane-runtime
  getter/setter methodsets.
- Package-level GoDoc/comment markers go in a file named `doc.go` — some
  codegen tools only check that filename, not `groupversion_info.go`.
- Add `GroupVersionKind` convenience vars per kind (typically in
  `register.go` or `groupversion_info.go`):
  ```go
  var (
  	FavouriteDBInstanceKind             = reflect.TypeOf(FavouriteDBInstance{}).Name()
  	FavouriteDBInstanceKindAPIVersion   = FavouriteDBInstanceKind + "." + GroupVersion.String()
  	FavouriteDBInstanceGroupVersionKind = GroupVersion.WithKind(FavouriteDBInstanceKind)
  )
  ```
- Use `angryjet` (from `crossplane-tools`) via `//go:generate` to generate
  the interface-satisfying getters/setters, rather than hand-writing them.
- Consider opening a **draft PR** once the API types are defined, before
  starting the controller, and ask a maintainer for early feedback.

## The controller pattern

Most managed resource controllers wrap `managed.NewReconciler` around an
`ExternalConnecter`/`ExternalClient` pair:

- **`Connect`** — given the managed resource, fetch the referenced
  `ProviderConfig` and its credentials Secret, then construct and return a
  client for the external API.
- **`Observe`** — call the external API, report whether the resource
  exists, whether it's up to date, and any connection details to publish.
  Copy output-only fields into `Status` here.
- **`Create`** — called only if `Observe` reported the resource doesn't
  exist. Must not error if the resource turns out to already exist —
  squash that specific error (`resource.Ignore(isExists, err)`) rather than
  risk "adopting" an unrelated existing resource.
- **`Update`** — called only if `Observe` reported the resource is not up
  to date.
- **`Delete`** — called when a managed resource with the `Delete` deletion
  policy (the default) is deleted. Must not error if the external resource
  is already gone (`resource.Ignore(isNotFound, err)`).

Skeleton:

```go
func (c *connecter) Connect(ctx context.Context, mg resource.Managed) (managed.ExternalClient, error) {
	i, ok := mg.(*v1alpha3.FavouriteDBInstance)
	if !ok {
		return nil, errors.New("managed resource is not a FavouriteDBInstance")
	}

	p := &fcpv1alpha3.Provider{}
	if err := c.client.Get(ctx, meta.NamespacedNameOf(i.Spec.ProviderReference), p); err != nil {
		return nil, errors.Wrap(err, "cannot get Provider")
	}

	s := &corev1.Secret{}
	n := types.NamespacedName{Namespace: p.Namespace, Name: p.Spec.Secret.Name}
	if err := c.client.Get(ctx, n, s); err != nil {
		return nil, errors.Wrap(err, "cannot get Provider secret")
	}

	client, err := database.NewClient(ctx, s.Data[p.Spec.Secret.Key])
	return &external{client: client}, errors.Wrap(err, "cannot create client")
}
```

Wire it up:

```go
func (c *FavouriteDBInstanceController) SetupWithManager(mgr ctrl.Manager) error {
	return ctrl.NewControllerManagedBy(mgr).
		Named(strings.ToLower(fmt.Sprintf("%s.%s", v1alpha3.FavouriteDBInstanceKind, v1alpha3.Group))).
		For(&v1alpha3.FavouriteDBInstance{}).
		Complete(managed.NewReconciler(mgr,
			resource.ManagedKind(v1alpha3.FavouriteDBInstanceGroupVersionKind),
			managed.WithExternalConnecter(&connecter{client: mgr.GetClient()})))
}
```

For readiness/condition semantics, see the "Conditions and events" section
of the main `SKILL.md`.

## Testing

Table-driven tests using Go's standard `testing` package — **do not** add
or proliferate Ginkgo-based tests, even though controller-runtime's own
ecosystem often uses Ginkgo elsewhere. This is called out explicitly in the
provider development guide as a project-wide norm.

## Further reading

- "Managed Resource API Patterns" one-pager:
  `crossplane/crossplane/design/one-pager-managed-resource-api-design.md`
- Naming conventions for external-resource-name fields are still an open
  discussion: `crossplane/crossplane#624`.
- Avoiding "adopting" unrelated existing resources:
  `crossplane/crossplane-runtime#27`.

[provider-dev]: https://github.com/crossplane/crossplane/blob/main/contributing/guide-provider-development.md
[provider-template]: https://github.com/crossplane/provider-template
[Upjet]: https://github.com/crossplane/upjet
