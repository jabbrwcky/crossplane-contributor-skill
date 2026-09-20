# AI contribution policy, in depth

Source: [Crossplane AI Contribution Policy][ai-policy]. Applies org-wide to
`github.com/crossplane` and `github.com/crossplane-contrib`, the same scope
as `references/governance.md`.

## AI usage is assumed, disclosure isn't required

The maintainers use AI coding tools daily and expect most contributors do
too, so no disclosure is required. But they explicitly **discourage
agent-authorship trailers** (e.g. a `Co-authored-by` line for an AI tool) in
commits — it pollutes contributor statistics. Omit them here even if your
own default workflow normally adds one.

## Own and understand your contribution

You're responsible for the entirety of your submission regardless of
tooling:

- **Understand every line you're changing.** If you can't explain a
  change's purpose and impact without the agent's help, it isn't ready to
  submit.
- **Think and verify critically**, on the initial PR and on every update in
  response to feedback — don't let the maintainer team be the first humans
  to critique or test the change.
- **Use the PR/issue templates** and actually fill in the checklist. A PR
  missing the template's checklist reads as evidence an agent opened it
  unsupervised.
- **Don't blame the agent.** "My agent wrote that" doesn't answer a review
  comment — you're expected to understand the change more deeply than that.

## Talk to maintainers yourself

- **Share your own reasoning** — why you framed the problem this way, what
  you tried, what you're unsure about — not a summary an agent generated
  after the fact.
- **Keep issue/PR text at a normal level of detail.** Agents tend to
  over-explain: narrating rejected alternatives or justifying correctness
  to the reviewer. That reasoning belongs in the PR description if it
  belongs anywhere — not as code comments, which should explain non-obvious
  *why*, not defend the change to a reviewer.
- **Write it yourself.** Tools to help find the right words (e.g. for a
  non-native speaker) are fine; pasting clearly-generated prose while
  engaging with a maintainer is not.
- **Review with your own judgment.** Using AI to help you understand a PR
  you're reviewing is fine; posting an agent's output as your review of
  someone else's PR is not — Crossplane already runs its own automated AI
  review.

## Respect maintainer time

- **Discuss non-trivial changes first** — open an issue and align on
  direction before implementing, to avoid spending review cycles on
  something that won't be accepted. Not a hard requirement, but expected
  for anything beyond a clearly-scoped fix.
- **Keep changes small and focused.**
- **Don't flood the queue** — avoid opening new PRs while you already have
  unmerged ones open, especially as a new contributor without an
  established trust relationship yet.

## Enforcement

Maintainers may close PRs/issues that don't meet the spirit of this policy,
without a detailed explanation first, and may block contributors who
continue to violate it. As a CNCF/Linux Foundation project, Crossplane is
also subject to the LF's [Generative AI policy][lf-ai-policy].

[ai-policy]: https://github.com/crossplane/crossplane/blob/main/AI_POLICY.md
[lf-ai-policy]: https://www.linuxfoundation.org/legal/generative-ai
