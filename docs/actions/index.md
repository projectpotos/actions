# Composite Actions

Step-level building blocks for repository-specific jobs that can't be a shared workflow
(a pytest matrix, a custom uv command, required-check aggregation):

| Action | Description |
|---|---|
| [setup-uv](setup-uv.md) | Checkout + install uv, ready for `uv run` |
| [all-green](all-green.md) | Aggregate required-check job |

GitHub materializes remotely referenced composite actions outside the workspace, so they
never interfere with linters scanning your repository tree.

Note that composite actions cannot be referenced relatively from reusable workflows
(step-level `./` paths resolve against the *consumer's* checkout). Anything that is a
complete job lives in a [reusable workflow](../workflows/index.md) instead.
