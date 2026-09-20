# crossplane-contrib governance and process

Source: `crossplane/provider-template/PROVIDER_CHECKLIST.md` and
`crossplane/crossplane/GOVERNANCE.md`. These apply specifically to
providers/extensions hosted under the `crossplane-contrib` org — they're
extra obligations on top of everything in `SKILL.md` and
`coding-style.md`/`provider-development.md`, not a replacement for them.
`crossplane-contrib` repos are "effectively sub-projects of Crossplane and
must follow all the same rules and policies" as `crossplane` core.

## Two-tier governance

- **Steering Committee** — org-wide, 5 seats, 2-year staggered terms,
  elected by Condorcet/IRV vote. No single organization (or conglomerate of
  affiliated ones) may hold more than 2 seats. Only gets involved for
  changes with broad cross-repo/architectural impact, or to bootstrap/veto
  in specific cases below.
- **Per-repo maintainer team** — each repo (including each
  crossplane-contrib provider) has its own maintainers, listed in that
  repo's `OWNERS.md`. Maintainers approve and merge their own repo's PRs,
  manage their own maintainer list, and triage their own issues
  independently.

## Repo naming and required files

- Provider repos are named `provider-<name>` (e.g. `provider-aws`,
  `provider-kubernetes`).
- Required at repo root: descriptive `README.md`, Apache 2.0 `LICENSE`, DCO,
  CNCF Code of Conduct, up-to-date `OWNERS.md`, `hack/boilerplate.go.txt`
  (correct license header for codegen), an `examples/` directory with a
  `ProviderConfig` example plus one example per resource, issue templates,
  and a PR template.
- Docs must cover: how to install the provider, how to contribute, and how
  to authenticate to the backend API (including creating the
  `ProviderConfig` secret).

## Build and release

- Use the shared `crossplane/build` submodule for a consistent Makefile-driven
  build across the ecosystem, plus a Go linter config.
- Package config lives at `package/crossplane.yaml`.
- Providers generally follow Crossplane's own release process — most use
  GitHub Actions to build/tag/promote releases, reusing workflows from
  `provider-template/.github/workflows`.
- Build artifacts (Crossplane packages / OCI images) **must be published to
  `ghcr.io/crossplane-contrib`** — the neutral, community-owned registry.
  `xpkg.crossplane.io` is a Scarf gateway in front of it for adoption
  metrics; projects may additionally publish elsewhere, but ghcr.io is the
  baseline.
- GitHub org-scoped secrets for pushing to shared registries are granted by
  a crossplane-contrib administrator/steering-committee member —
  individual providers don't self-provision registry credentials.

## API group convention

New crossplane-contrib extension projects should use a `*.crossplane.io`
API group. Only providers that started in a different org before being
contributed to crossplane-contrib may keep a pre-existing, different API
group (so as not to break existing users on upgrade).

## Bootstrapping a brand-new provider

Open an issue in `crossplane/org` naming:

- The extension and its purpose/scope.
- The initial maintainer team (GitHub usernames).
- That team's long-term commitment to maintaining it.

The steering committee reviews and approves before the repo is created and
initial maintainers are added — this replaces the normal maintainer-trial
process below, since there's no existing team yet to vouch for a new one.

## Becoming a maintainer on an existing provider

- Demonstrate leadership/domain expertise and proficiency in the languages/
  tools the project uses.
- Go through an informal ~2–3 month trial: submit non-trivial PRs, do code
  reviews, participate in issue triage.
- The existing maintainer team votes you in — simple majority for most
  decisions, but a **2/3 supermajority specifically for maintainer
  additions/removals** and for exercising a veto.

## Maintainer responsibilities

Participate in design discussion; ensure changes meet established
standards; contribute non-trivial PRs themselves; review code; triage
issues regularly; keep PRs moving (or close stale ones); monitor Slack;
attend community meetings; commit roughly 25%+ of their time to the
project.

## Other ongoing obligations

- Follow all CNCF policies and "project health" guidelines.
- Keep the maintainer list synchronized in **three** places: the repo's
  `OWNERS.md`, the GitHub team granting merge permissions on the repo, and
  the central `crossplane/.project` maintainer list
  (`maintainers.yaml`).
- Keep the project's entry current in the public "Community Extension
  Projects" list on docs.crossplane.io.

## Archival policy

If a project falls out of compliance with health/policy guidelines and the
issue can't be resolved with its maintainer team, the steering committee
can archive the repo. Public crossplane.io/docs references to it are then
removed. The public docs/website only reference healthy, compliant,
non-archived, non-vendor-owned crossplane-contrib projects.
