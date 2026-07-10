# Security: Permissions

These reusable workflows enforce
[least-privilege](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions)
by explicitly declaring the minimum `permissions` each workflow job requires. GitHub Actions
enforces the intersection of caller and callee permissions, so the effective permissions for
a called workflow are **no more** than what the calling job grants.

## Implementing Least Access

1. **Restrict default token permissions** in your repository's Settings → Actions →
   General → Workflow permissions. Select **"Read repository contents and packages
   permissions"**.

2. **Set `permissions: {}`** at the top of every calling workflow, then grant only what
   each job needs at the job level. Every example in this documentation already follows
   this pattern.

3. **Keep job-level permissions tightly scoped.** The table below lists the minimum
   permissions each reusable workflow requires. Only grant what is listed.

## Permissions Reference

| Reusable Workflow | Required `permissions` |
|---|---|
| `release-ansible-collection.yaml` | `contents: read` |
| `release-container.yaml` | `contents: read`, `packages: write`, `id-token: write`, `attestations: write` |
| `release-zensical.yaml` | `contents: read`, `pages: write`, `id-token: write` |
| `semantic-release.yaml` | `contents: write` (creates tags and releases) |
| `test-ansible-collection.yaml` | `contents: read` |
| `test-lint.yaml` | `contents: read` |

Composite actions:

| Composite Action | Required `permissions` |
|---|---|
| `setup-uv` | `contents: read` (checks out the repository) |
| `all-green` | none |

For further reading see GitHub's
[Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions)
guide.
