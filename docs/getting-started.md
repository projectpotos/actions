# Getting Started

## Usage

The examples on this site use `@v0.0.0` as the target version. You **must** replace that
with the full commit SHA of the current release of this repository, keeping the version tag
as a comment:

```yaml
uses: projectpotos/actions/.github/workflows/test-lint.yaml@<full-commit-sha> # vX.Y.Z
```

To keep the pins up to date automatically, create a `.github/dependabot.yml` file:

```yaml title=".github/dependabot.yml"
---
version: 2
updates:
  - package-ecosystem: github-actions
    directory: "/"
    schedule:
      interval: daily
    commit-message:
      prefix: "chore(ci): "
    cooldown:
      default-days: 7
```

Dependabot updates the commit SHA and the version comment together, for example
`@abc1234 # v3` → `@def5678 # v4`.

## Security

These reusable workflows enforce
[least-privilege](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions)
by explicitly declaring the minimum `permissions` each workflow job requires.

To implement least access in your repositories:

1. **Restrict default token permissions** in Settings → Actions → General → Workflow
   permissions. Select **"Read repository contents and packages permissions"**.

2. **Set `permissions: {}`** at the top of every calling workflow to start from a baseline
   of no permissions, then grant only what each job needs at the job level. Every example in
   this documentation follows this pattern.

3. **Pin to commit SHAs** rather than mutable version tags — both for this repository and
   for any third-party actions you use directly. Every workflow in this library pins its
   third-party actions the same way.

For potos organization repositories, `allowed_actions` and required status checks are
managed centrally via OpenTofu in the repo config repository — new workflow adoptions may
need an `allowed_actions` update there for the third-party actions used *inside* the called
workflows (same-org references themselves are always allowed).

See [Permissions](security/permissions.md) for what each workflow requires and
[Supply Chain](security/supply-chain.md) for the full pinning policy.

## Required status checks

All test workflows are designed to aggregate into a single required check via the
[all-green](actions/all-green.md) composite action:

```yaml
  all_green:
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

Skipped jobs are treated as OK (only `failure` and `cancelled` fail the check), so
conditional jobs don't break required status checks.
