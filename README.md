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

## License

Apache-2.0 — see [LICENSE](LICENSE).

[contributing]: https://github.com/crossplane/crossplane/blob/main/contributing/README.md
[provider-dev]: https://github.com/crossplane/crossplane/blob/main/contributing/guide-provider-development.md
