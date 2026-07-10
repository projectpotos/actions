# Ansible Collections: Release

Releases an Ansible collection to [Ansible Galaxy](https://galaxy.ansible.com/): resolves
the version from the pushed tag (or a manual input), bumps `galaxy.yml`, regenerates the
changelog with `antsibull-changelog`, builds and publishes the collection, commits the
release changes back to the repository, and uploads the tarball as a workflow artifact.

## Usage

Create a `.github/workflows/release.yaml` file. The repository needs a `release`
[environment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
with the `ANSIBLE_GALAXY_API_KEY` secret:

```yaml title=".github/workflows/release.yaml"
---
name: Release

on:
  push:
    tags: ["v*"]
  workflow_dispatch:
    inputs:
      version:
        description: "Version to release (without leading v), e.g. 1.2.3"
        required: true
        type: string

permissions: {} # (1)

concurrency:
  group: release-${{ github.ref }}
  cancel-in-progress: false

jobs:
  release:
    permissions:
      contents: write # (2)
    uses: projectpotos/actions/.github/workflows/release-ansible-collection.yaml@v0.0.0
    with:
      version: ${{ inputs.version }} # (3)
    secrets:
      ANSIBLE_GALAXY_API_KEY: ${{ secrets.ANSIBLE_GALAXY_API_KEY }}
```

1. Deny all permissions at the workflow level as a secure baseline.
2. Required to push the release commit (galaxy.yml + changelog) back to the repository.
3. Empty on tag pushes — the version is then derived from the tag name.

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `version` | Version to release, without leading `v` (empty = derive from the pushed tag) | No | `""` |
| `release-branch` | Branch to push the release commit back to | No | `main` |

## Secrets

| Secret | Description | Required |
|---|---|---|
| `ANSIBLE_GALAXY_API_KEY` | API key used to publish the collection on Ansible Galaxy | **Yes** |
