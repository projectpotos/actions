# Composite Actions: setup-uv

Checks out the repository (`persist-credentials: false`) and installs
[uv](https://docs.astral.sh/uv/), ready to run project tooling (pytest, molecule,
antsibull-changelog, ...). Replaces the two boilerplate steps at the start of nearly every
job.

## Usage

```yaml title=".github/workflows/tests.yaml (excerpt)"
jobs:
  pytest:
    name: pytest (py${{ matrix.python-version }})
    runs-on: ubuntu-latest
    timeout-minutes: 10
    permissions:
      contents: read # (1)
    strategy:
      fail-fast: false
      matrix:
        python-version: ["3.12", "3.13", "3.14"]
    steps:
      - uses: projectpotos/actions/setup-uv@v0.0.0
        with:
          python-version: ${{ matrix.python-version }} # (2)
      - run: uv run --frozen --group test pytest tests/ -v
```

1. Required to check out code.
2. Omit to use the project's default Python version.

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `python-version` | Python version to install (empty = project default) | No | `""` |
| `enable-cache` | Enable the uv cache | No | `true` |
| `fetch-depth` | Number of commits to fetch (`0` = full history) | No | `1` |
