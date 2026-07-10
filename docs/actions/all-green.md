# Composite Actions: all-green

Verifies that no required job failed or was cancelled. Use it as the single job behind the
`all_green` required status check: pass the full `needs` context and it fails if any needed
job has result `failure` or `cancelled`. **Skipped jobs are treated as OK**, so conditional
jobs don't break required status checks.

## Usage

```yaml title=".github/workflows/tests.yaml (excerpt)"
jobs:
  # ... the actual test jobs ...

  all_green:
    if: ${{ always() }} # (1)
    needs: [lint, pytest] # (2)
    runs-on: ubuntu-latest
    timeout-minutes: 5
    permissions: {} # (3)
    steps:
      - uses: projectpotos/actions/all-green@v0.0.0
        with:
          needs: ${{ toJSON(needs) }} # (4)
```

1. Run even when a needed job failed — otherwise the check would be skipped, not failed.
2. List **every** job that must pass.
3. The action needs no checkout and no token.
4. Pass the whole `needs` context; the action inspects each job's result with jq.

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `needs` | The needs context of the aggregating job: `${{ toJSON(needs) }}` | **Yes** | — |
