# Potos GitHub Actions

These are the [reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
and composite actions that Project Potos uses for CI/CD ♻️

📖 **Full documentation: [projectpotos.github.io/actions](https://projectpotos.github.io/actions/)**

## Quick Examples

Replace `@v0.0.0` with the full commit SHA of the current release (version tag as comment);
Dependabot keeps the pins fresh.

### Ansible Collection CI

```yaml title=".github/workflows/tests.yaml"
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

permissions: {}

jobs:
  ci:
    permissions:
      contents: read
    uses: projectpotos/actions/.github/workflows/test-ansible-collection.yaml@v0.0.0
    with:
      molecule-scenarios: '["basics", "periodic"]'

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

### Container Image Release

```yaml title=".github/workflows/release.yaml"
name: Release

on:
  pull_request:
  push:
    branches: [main]
    tags: ["v*.*.*"]

permissions: {}

jobs:
  build:
    permissions:
      contents: read
      packages: write
      id-token: write
      attestations: write
    uses: projectpotos/actions/.github/workflows/release-container.yaml@v0.0.0
    with:
      tags: |
        type=edge,branch=main
        type=semver,pattern={{version}}
      labels: |
        org.opencontainers.image.title=<name>
```

### Lint

```yaml title=".github/workflows/lint.yaml"
name: Lint

on:
  pull_request:
  push:
    branches: [main]

permissions: {}

jobs:
  lint:
    permissions:
      contents: read
    uses: projectpotos/actions/.github/workflows/test-lint.yaml@v0.0.0
```

For all available workflows and their full configuration options, see the
[documentation](https://projectpotos.github.io/actions/).

## License

These reusable workflows are free software: you can redistribute them and/or modify them
under the terms of the GNU General Public License as published by the Free Software
Foundation, version 3 of the License, or (at your option) any later version.

## Copyright

Copyright (c) 2026 [Project Potos](https://potos.dev)
