# Composite Actions

Step-level building blocks for repository-specific jobs that can't be a shared workflow:

| Action | Description |
|---|---|
| [setup-uv](setup-uv.md) | Checkout + install uv, ready for `uv run` |
| [all-green](all-green.md) | Aggregate required-check job |


Note that composite actions cannot be referenced relatively from reusable workflows
(step-level `./` paths resolve against the *consumer's* checkout). Anything that is a
complete job lives in a [reusable workflow](../workflows/index.md) instead.
