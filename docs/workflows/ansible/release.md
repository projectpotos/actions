# Ansible Collections: Release

Publishes an Ansible collection to [Ansible Galaxy](https://galaxy.ansible.com/) when a
version tag is pushed. The workflow is for **publishing only**: it verifies that the tag matches
the `galaxy.yml` version, builds the collection, publishes it, and uploads the tarball as a
workflow artifact. It needs no write access to the repository.

The version bump and changelog must be adjusted before calling this workflow.

If the tag does not match `galaxy.yml` on the tagged commit, the workflow fails before publishing anything.

## Usage

Create a `.github/workflows/release.yaml` file. The repository needs a `release`
[environment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
with the `ANSIBLE_GALAXY_API_KEY` secret.

```yaml title=".github/workflows/release.yaml"
---
name: Release

on:
  push:
    tags: ["v*"]

permissions: {} # (1)

concurrency:
  group: release-${{ github.ref }}
  cancel-in-progress: false

jobs:
  release:
    permissions:
      contents: read # (2)
    uses: projectpotos/actions/.github/workflows/release-ansible-collection.yaml@v0.0.0
    secrets:
      ANSIBLE_GALAXY_API_KEY: ${{ secrets.ANSIBLE_GALAXY_API_KEY }}
```

1. Deny all permissions at the workflow level as a secure baseline.
2. Publishing needs no write access — the version bump should land via a release PR.

## Inputs

None — the version is derived from the pushed tag and verified against `galaxy.yml`; the
artifact name is derived from the collection's namespace and name in `galaxy.yml`.

## Secrets

| Secret | Description | Required |
|---|---|---|
| `ANSIBLE_GALAXY_API_KEY` | API key used to publish the collection on Ansible Galaxy | **Yes** |