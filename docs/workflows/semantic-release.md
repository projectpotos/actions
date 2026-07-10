# Semantic Release

Automates versioning with [go-semantic-release](https://go-semantic-release.xyz/): on every
push to `main` it parses the [Conventional Commits](https://www.conventionalcommits.org/)
since the last release and creates the next tag and GitHub release automatically.

This workflow also releases this repository itself.

## Usage

Create a `.github/workflows/semantic-release.yaml` file:

```yaml title=".github/workflows/semantic-release.yaml"
---
name: Semantic Release

on:
  push:
    branches: [main]

permissions: {} # (1)

jobs:
  semantic-release:
    permissions:
      contents: write # (2)
    uses: projectpotos/actions/.github/workflows/semantic-release.yaml@v0.0.0
```

1. Deny all permissions at the workflow level as a secure baseline.
2. Required to create tags and GitHub releases.

Note: releases created with the default `GITHUB_TOKEN` do not trigger other workflows
(such as a tag-triggered release workflow). If a release must trigger follow-up workflows,
run those on the release event from a manually dispatched workflow, or use a dedicated app
token.

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `dry` | Dry-run mode: determine the next version without creating a release | No | `false` |
