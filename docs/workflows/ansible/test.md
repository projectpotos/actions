# Ansible Collections: Test

Full CI for an Ansible collection: the upstream
[`ansible/ansible-content-actions`](https://github.com/ansible/ansible-content-actions)
suite (changelog check on PRs, build-import, ansible-lint, sanity, unit tests), the shared
[lint suite](../lint.md), `antsibull-changelog lint`, and an optional molecule matrix.

## Usage

Create a `.github/workflows/tests.yaml` file:

```yaml title=".github/workflows/tests.yaml"
---
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
  workflow_dispatch:

permissions: {} # (1)

concurrency:
  group: ci-${{ github.head_ref || github.run_id }}
  cancel-in-progress: true

jobs:
  ci:
    permissions:
      contents: read # (2)
    uses: projectpotos/actions/.github/workflows/test-ansible-collection.yaml@v0.0.0
    with:
      molecule-scenarios: >- # (3)
        ["basics", "firstboot", "firstboot_luks", "firstboot_openbao",
         "firstboot_playbook", "periodic"]

  all_green: # (4)
    if: ${{ always() }}
    needs: [ci]
    runs-on: ubuntu-latest
    timeout-minutes: 5
    permissions: {}
    steps:
      - uses: projectpotos/actions/all-green@v0.0.0
        with:
          needs: ${{ toJSON(needs) }}
```

1. Deny all permissions at the workflow level as a secure baseline.
2. Required to check out code; nothing in the CI suite writes.
3. The molecule scenarios of your collection, as a JSON array. Omit to skip molecule.
4. Single required status check; see [all-green](../../actions/all-green.md).

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `molecule-scenarios` | JSON array of molecule scenarios to run (empty = skip molecule) | No | `[]` |
| `molecule-distros` | JSON array of distros for the molecule matrix | No | `["fedora44"]` |
| `unit-source-matrix-exclude` | JSON `matrix_exclude` for the unit-source job | No | recent python/ansible versions only |
| `ruff` | Run ruff check + format | No | `true` |
| `ruff-group` | Optional uv dependency group that provides ruff | No | `""` |
| `changelog-lint` | Run antsibull-changelog lint | No | `true` |
