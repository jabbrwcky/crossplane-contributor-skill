---
name: crossplane-contributor
description: >
  Guidance for contributing code to crossplane/crossplane or any
  crossplane-contrib provider repository. Covers commit hygiene, DCO
  sign-off, PR structure, coding style, and provider-specific API/controller
  conventions. Use this skill before opening a PR to a crossplane or
  crossplane-contrib repo, when splitting or reviewing commits for such a
  PR, when authoring a new managed resource or ProviderConfig, or when
  asked to check compliance with crossplane's contributing guide.
---

# Crossplane contributor

Distilled from the official [crossplane contributing guide][contributing]
and [provider development guide][provider-dev]. Applies to
`github.com/crossplane` and `github.com/crossplane-contrib` repositories
alike — crossplane-contrib providers follow the same rules, plus a few
extra process obligations (see `references/governance.md`).

For anything not covered here, defer to the linked upstream docs — they are
the source of truth; this skill is a working summary, not a replacement.

## Before writing code

- Check for an existing GitHub issue describing the change; if none exists,
  open one. This lets maintainers steer you before you invest in an
  approach that would be rejected or reworked.
- **New feature** (not a bug fix)? It can only land during the active
  development window of a release cycle — check the release cycle docs if
  timing is tight.
- **Brand-new provider repository**? This needs a `crossplane/org` issue
  naming the extension, its scope, and the initial maintainer team, for
  steering-committee approval before a repo is created. See
  `references/governance.md`.

## Commit hygiene

- **Sign off every commit**: `git commit -s` (DCO). A bot enforces this on
  every PR; there is no way to skip it.
- **One logical change per commit.** Commits should "tell a story" for a
  reviewer — not read as a diff dump. If a change naturally splits into
  layers (e.g. new package → callers wired to it → API surface → docs),
  make each layer its own commit.
- **Keep documentation changes in their own commit**, separate from the
  code they document.
- **Never use placeholder messages** like "Address review feedback" —
  amend or interactively rebase so the final history reads cleanly.
- **Don't force-push to address review feedback** once a reviewer has
  started commenting — that erases the record of what they saw. It's fine
  to rewrite history (squash, reorder, split) *before* any review has
  started, or once a PR is approved but not yet merged.
- Use `github.com/pkg/errors` or `crossplane-runtime/pkg/errors` to wrap
  errors with inline context strings, not error constants — see
  `references/coding-style.md` for why and how.

## Coding style checklist

Full detail and examples in `references/coding-style.md`. Quick checklist:

- [ ] Return early; avoid `else` (see the Go idiom: if the `if` body ends
      in `return`/`break`/`continue`/`goto`, omit the `else`).
- [ ] Short variable names for short-lived, unambiguous locals (`c`, `i`,
      `r`); descriptive names only when scope or type doesn't already make
      the meaning obvious.
- [ ] Don't wrap function signatures across lines — if you need many
      optional parameters, use a functional-options pattern instead.
- [ ] Every `//nolint:<linter>` has the tightest possible scope and an
      inline comment explaining *why*.
- [ ] Test error **properties**, not error strings — use
      `cmpopts.EquateErrors()` (or `cmpopts.AnyError` when you only care
      that *an* error occurred).
- [ ] Table-driven tests only. No Ginkgo, Gomega, or Testify — PRs
      introducing them get review pushback.
- [ ] Test case names and function names are PascalCase, no underscores or
      spaces.
- [ ] Table cases include a `reason` field (a human sentence explaining
      what's being verified) that's printed on failure.
- [ ] Scope `err` as narrowly as possible (declare inside the `if`, not at
      function scope) to avoid accidental shadowing/reuse as code evolves.

## Provider API conventions

Full detail in `references/provider-development.md`. Quick reference:

- Start a **brand-new provider** from the [provider-template] repo, not
  bare kubebuilder scaffolding — it already encodes these conventions.
  Adding a resource to an **existing** provider? Copying and adapting a
  sibling resource in the same repo is often faster and safer than
  regenerating from kubebuilder.
- A managed resource type must: satisfy `resource.Managed`; embed
  `xpv1.ResourceStatus` in `Status`; embed `xpv1.ResourceSpec` **and** a
  `Parameters` struct in `Spec`; carry `+kubebuilder:subresource:status`
  and `+kubebuilder:resource:scope=Cluster`.
- `Parameters` should be a **high-fidelity** mirror of the external API's
  writeable fields (only reshaped where Kubernetes API conventions demand
  it, e.g. `snake_case` → `camelCase`). Output-only fields belong in
  `Status`, never `Parameters`.
- `ProviderConfig` types must be named exactly `ProviderConfig`, embed
  `xpv1.ProviderSpec`, and be cluster-scoped.
- Package-level GoDoc and comment markers go in `doc.go` — some codegen
  tools only look there, not in `groupversion_info.go`.
- Use `angryjet` (`//go:generate`) to generate the getter/setter boilerplate
  crossplane-runtime interfaces require, instead of hand-writing it.
- After any API type change: run `make reviewable` (or the repo's Makefile
  equivalent) to regenerate CRDs and deepcopy/methodset code before
  committing.
- `Create` must not error if the external resource already exists; `Delete`
  must not error if it's already gone (`resource.Ignore(isNotFound, err)`).
  Don't silently "adopt" an unrelated existing external resource.
- Document every field with GoDoc written for someone running
  `kubectl explain` or reading generated API docs — not for someone reading
  the Go source.

## Conditions and events

- Condition `reason` is machine-readable CamelCase; `message` is a stable,
  plain-English sentence — no timestamps, no relative time, no
  non-deterministic ordering (sort maps before rendering).
- Never surface a transient error (e.g. an apiserver conflict) as a
  condition or event — requeue silently instead.
- Emit an event only when something **happened** (an action succeeded or
  failed), never for "nothing changed" or "about to do X".

## PR checklist

Most repos' PR templates ask for roughly this:

- [ ] Read and followed the contribution process.
- [ ] Ran `make reviewable` (or equivalent) — build, generate, lint, test
      all pass.
- [ ] `Fixes #<issue>` if applicable.
- [ ] Description explains *why*, not just *what*.
- [ ] Test plan describes what was tested and how.

Leave a checklist box unticked if you're genuinely unsure whether it
applies — that's a deliberate signal for the reviewer to weigh in, not a
failure.

## Reference files

- `references/coding-style.md` — full coding-style rules with before/after
  examples (errors, tests, function signatures, variable naming).
- `references/provider-development.md` — managed resource / ProviderConfig
  / controller pattern in more depth, with a worked example.
- `references/governance.md` — crossplane-contrib-specific process: repo
  bootstrapping, OWNERS.md, registry publishing, maintainer trial, archival
  policy.

[contributing]: https://github.com/crossplane/crossplane/blob/main/contributing/README.md
[provider-dev]: https://github.com/crossplane/crossplane/blob/main/contributing/guide-provider-development.md
[provider-template]: https://github.com/crossplane/provider-template
