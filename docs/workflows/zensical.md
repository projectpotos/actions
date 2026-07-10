# Zensical

Builds a [Zensical](https://zensical.org/) documentation site and deploys it to GitHub
Pages. The repository needs a `zensical.toml` at its root and GitHub Pages configured for
the "GitHub Actions" source.

This workflow also deploys this repository's own documentation on every push to `main`.

## Usage

Create a `.github/workflows/docs.yaml` file:

```yaml title=".github/workflows/docs.yaml"
---
name: Documentation

on:
  push:
    branches: [main]
  pull_request: # (1)

permissions: {} # (2)

concurrency:
  group: docs-${{ github.ref }}
  cancel-in-progress: false

jobs:
  docs:
    permissions:
      contents: read # (3)
      pages: write # (4)
      id-token: write # (5)
    uses: projectpotos/actions/.github/workflows/release-zensical.yaml@v0.0.0
    with:
      deploy: ${{ github.event_name != 'pull_request' }} # (6)
```

1. On pull requests the site is built (validating the docs) but not deployed.
2. Deny all permissions at the workflow level as a secure baseline.
3. Required to check out the documentation source.
4. Required to deploy the artifact to GitHub Pages (gated on the `deploy` input).
5. Required to authenticate the Pages deployment via OIDC (gated on the `deploy` input).
6. Build-only on PRs; build and deploy on pushes to `main`.

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `deploy` | Deploy the built site to GitHub Pages | No | `true` |
