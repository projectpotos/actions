# Container Images: Release

Builds a container image with Buildx, pushes it to a registry, and
attaches a [build provenance attestation](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations).
On `pull_request` events the image is built but not pushed or attested.

## Usage

Create a `.github/workflows/release.yaml` file:

```yaml title=".github/workflows/release.yaml"
---
name: Release

on:
  push:
    branches: [main]
    tags: ["v*.*.*"]
  pull_request:
  workflow_dispatch:

permissions: {} # (1)

jobs:
  build:
    permissions:
      contents: read # (2)
      packages: write # (3)
      id-token: write # (4)
      attestations: write # (5)
    uses: projectpotos/actions/.github/workflows/release-container.yaml@v0.0.0
    with:
      tags: | # (6)
        type=edge,branch=main
        type=semver,pattern={{version}}
        type=semver,pattern={{major}}.{{minor}}
        type=semver,pattern={{major}}
        type=sha
      labels: |
        org.opencontainers.image.title=<name>
        org.opencontainers.image.description=<description>
        org.opencontainers.image.licenses=GPL-3.0-or-later
```

1. Deny all permissions at the workflow level as a secure baseline.
2. Required to check out code.
3. Required to push the built image to the registry (gated on non-PR events).
4. Required for keyless signing of the provenance attestation via GitHub OIDC.
5. Required to write the build-provenance attestation.
6. A [docker/metadata-action](https://github.com/docker/metadata-action#tags-input)
   tag specification.

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `tags` | Tag specification for docker/metadata-action (multiline) | **Yes** | — |
| `labels` | OCI labels for docker/metadata-action (multiline) | No | `""` |
| `platforms` | Comma-separated target platforms | No | `linux/amd64` |
| `setup-qemu` | Set up QEMU for cross-platform builds | No | `false` |
| `registry` | Container registry to publish to | No | `ghcr.io` |
| `image-name` | Image name | No | calling repository |
| `context` | Context directory for the Docker build | No | `.` |
| `dockerfile` | Path to the Dockerfile, relative to the context | No | `Dockerfile` |
