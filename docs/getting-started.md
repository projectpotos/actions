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

If you need multiple workflows in the same repository, combine them as needed. Please add
an example to the docs if you use the same combination more than once.

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

4. **Immutable releases** — releases in this repository are [immutable](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository). Once a GitHub Release is published it cannot be edited or deleted. When you pin to a released tag or its underlying commit SHA you can be confident the content will never silently change.

See  for what each workflow requires and
[Supply Chain](security/supply-chain.md) for the full pinning policy.

See the [Permissions](security/permissions.md) page for a full reference of what each workflow requires, and the [Security](security/) section for the full security controls documentation.

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

## Issue and Pull Request Templates

To make managing your CI/CD configuration easier, add the issue templates and PR template
from this repository to your own project. They provide structured forms for reporting
workflow bugs, requesting updates, and tracking version changes.

Copy the files from `.github/ISSUE_TEMPLATE/` and `.github/PULL_REQUEST_TEMPLATE.md` in this
repository and adapt the workflow list to the ones you actually use.

See the [Contributing](contributing.md) guide for the full workflow lifecycle and how these
templates are used to maintain this library.
