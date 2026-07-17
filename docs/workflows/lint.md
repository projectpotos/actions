# Lint

The lint suite for all potos repositories. Some linters like yamllint, actionlint, and zizmor always
run, with the assumption that repository must at least have a workflow definition.

Depending on the projects needs ruff, hadolint, shellcheck, and shellcheck-jinja can be
enabled as well.

shellcheck-jinja lints shell scripts that only exist as Jinja2 templates (`*.sh.j2`):  
it renders every template with fixture data from the repository and runs shellcheck on the rendered output, using
[projectpotos/shellcheck-jinja](https://github.com/projectpotos/shellcheck-jinja).

Other workflows in this library (like
[test-ansible-collection](ansible/test.md)) embed this suite via a relative reference, so it
always runs at the same version as the calling workflow.

## Usage

Create a `.github/workflows/lint.yaml` file:

```yaml title=".github/workflows/lint.yaml"
---
name: Lint

on:
  pull_request:
  push:
    branches: [main]
  workflow_dispatch:

permissions: {} # (1)

concurrency:
  group: lint-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint:
    permissions:
      contents: read # (2)
    uses: projectpotos/actions/.github/workflows/test-lint.yaml@v0.0.0
    with:
      ruff: true # (3)
      ruff-group: lint
      hadolint: true
      shellcheck: true

  all_green:
    if: ${{ always() }}
    needs: [lint]
    runs-on: ubuntu-latest
    timeout-minutes: 5
    permissions: {}
    steps:
      - uses: projectpotos/actions/all-green@v0.0.0
        with:
          needs: ${{ toJSON(needs) }}
```

1. Deny all permissions at the workflow level as a secure baseline.
2. Required to check out code; the lint suite only reads.
3. Enable the opt-in linters that apply to your repository.

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `yamllint-mode` | `project` runs yamllint from the repo's locked uv project, `standalone` runs it via uvx | No | `project` |
| `ruff` | Run ruff check + format | No | `false` |
| `ruff-group` | Optional uv dependency group that provides ruff | No | `""` |
| `ruff-mode` | `project` runs ruff from the repo's locked uv project, `standalone` runs it via uvx | No | `project` |
| `hadolint` | Lint the Dockerfile with hadolint | No | `false` |
| `dockerfile` | Path to the Dockerfile for hadolint | No | `Dockerfile` |
| `shellcheck` | Run shellcheck over all shell scripts | No | `false` |
| `shellcheck-opts` | Extra options passed to shellcheck | No | `""` |
| `shellcheck-jinja` | Render `*.sh.j2` templates with fixture data and shellcheck the results | No | `false` |
| `shellcheck-jinja-group` | Optional uv dependency group that provides shellcheck-jinja | No | `""` |
| `shellcheck-jinja-fixtures` | Directory containing the shellcheck-jinja fixture files | No | `tests/shellcheck-fixtures` |
| `shellcheck-jinja-path` | Directory of the uv project the shell templates and fixtures live in | No | `.` |
| `zizmor-persona` | zizmor persona: `regular`, `pedantic`, or `auditor` | No | `regular` |
