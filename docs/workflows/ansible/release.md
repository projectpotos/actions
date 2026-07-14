# Ansible Collections: Release

Builds and optionally publishes an Ansible collection to [Galaxy](https://galaxy.ansible.com/).

## Usage

Create a `.github/workflows/release.yaml` file.

```yaml title=".github/workflows/release.yaml"
---
name: Release

on:
  push:
    tags: ["v*"]

permissions: {} # (1)

jobs:
  release:
    permissions:
      contents: read # (2)
    uses: projectpotos/actions/.github/workflows/release-ansible-collection.yaml@v0.0.0
    secrets:
      ANSIBLE_GALAXY_API_KEY: ${{ secrets.ANSIBLE_GALAXY_API_KEY }}
```

1. Deny all permissions at the workflow level as a secure baseline.
2. Publishing needs no write access to the repo directly.

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `publish` | Publish the built collection to Ansible Galaxy | No | `true` |
| `path` | Path to the collection root (directory containing `galaxy.yml`) | No | `.` |

## Secrets

| Secret | Description | Required |
|---|---|---|
| `ANSIBLE_GALAXY_API_KEY` | API key used to publish the collection on Ansible Galaxy | When `publish` is `true` |
