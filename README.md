# crossplane-contributor-skill

A [Claude Code](https://claude.com/claude-code) plugin that packages the
[crossplane contributing guide][contributing] and
[provider development guide][provider-dev] into a skill: guidance for
contributing to `crossplane/crossplane` or any `crossplane-contrib` provider
repository — commit hygiene, DCO sign-off, PR structure, coding style, and
provider-specific API/controller conventions.

It's a working summary derived from the upstream docs, not a mirror of
them — always defer to the linked upstream sources for anything this skill
doesn't cover or that has since changed.

## Install

```
/plugin marketplace add jabbrwcky/crossplane-contributor-skill
/plugin install crossplane-contributor@crossplane-contributor-skill
```

## What's in it

- `skills/crossplane-contributor/SKILL.md` — the core checklist: commit
  hygiene, coding style, provider API conventions, conditions/events, PR
  checklist.
- `skills/crossplane-contributor/references/coding-style.md` — full
  coding-style rules with before/after examples.
- `skills/crossplane-contributor/references/provider-development.md` —
  managed resource / `ProviderConfig` / controller pattern in depth, with a
  worked example.
- `skills/crossplane-contributor/references/governance.md` —
  crossplane-contrib-specific process: repo bootstrapping, `OWNERS.md`,
  registry publishing, maintainer trial, archival policy.
- `skills/crossplane-contributor/references/ai-policy.md` — Crossplane's
  AI contribution policy: ownership, authentic engagement, enforcement.

## Releasing

Bump `version` in `.claude-plugin/plugin.json` and push to `main`. CI takes it from there:

1. `tag-version.yml` detects the version change and pushes a `v<version>` tag.
2. `release.yml` runs on that tag push, rebuilds `crossplane-contributor.skill` from the current `skills/crossplane-contributor/` sources, and publishes a GitHub Release with it attached and a changelog compiled from the conventional commits since the last tag.

For Claude.ai, grab the `.skill` file from the release assets and upload it (there's no auto-update path there).

**Repo setup**: `tag-version.yml` pushes the tag with a PAT stored in the `RELEASE_TOKEN` secret, not the default `GITHUB_TOKEN` (tags pushed with the default token don't trigger other workflows), and then explicitly dispatches `release.yml` via the API rather than relying on its `push: tags` trigger firing on its own (that trigger is unreliable for tags created by a workflow). Add a fine-grained PAT with `Contents: Read and write` **and** `Actions: Read and write` on this repo as a repository secret named `RELEASE_TOKEN`.

## License

Apache-2.0 — see [LICENSE](LICENSE).

[contributing]: https://github.com/crossplane/crossplane/blob/main/contributing/README.md
[provider-dev]: https://github.com/crossplane/crossplane/blob/main/contributing/guide-provider-development.md
