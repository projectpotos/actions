# Potos GitHub Actions

These are the [reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
and composite actions that Project Potos uses for CI/CD ♻️

The goal is that **all pipelines come from this repository**: every potos project calls these
workflows instead of maintaining its own, so there is a single point for all external
dependencies — one place where third-party actions are selected, SHA-pinned, and kept
up to date.

## Where to go next

- [Getting Started](getting-started.md) — how to call the workflows and set up the security
  baseline in your repository
- [Workflows](workflows/index.md) — all reusable workflows by category
- [Composite Actions](actions/index.md) — step-level building blocks for repo-specific jobs
- [Security](security/index.md) — permissions reference and supply-chain controls
- [Contributing](contributing.md) — how to propose, change, and retire workflows
- [Workflow Analysis](analysis.md) — the analysis across potos repositories that this
  library is based on
