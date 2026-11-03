# PR Labels

Labels a pull request from its [Conventional Commits](https://www.conventionalcommits.org/)
title with the [release-drafter](https://github.com/release-drafter/release-drafter)
autolabeler and fails the check unless the pull request carries one of the labels the
collection changelog ([Draft Release](ansible/draft-release.md)) is generated from.

| Title prefix | Label | Version bump |
|---|---|---|
| `feat:`, `perf:` | `enhancement` | minor |
| `fix:` | `bug` | patch |
| `refactor:`, `style:` | `refactoring` | patch |
| `<type>!:` | `major` | major |
| `chore:`, `ci:`, `test:`, `build:`, `docs:` | `skip-changelog` | none, hidden from the notes |

Pull requests from forks run with a read-only token, so the autolabeler cannot label them;
a maintainer adds the label by hand.

## Usage

Create a `.github/workflows/pr-labels.yaml` file and a `.github/release-drafter.yml`
config (copy the one in this repository):

```yaml title=".github/workflows/pr-labels.yaml"
---
name: PR labels

on:
  pull_request:
    types: [opened, reopened, synchronize, labeled, unlabeled]

permissions: {} # (1)

jobs:
  pr-labels:
    permissions:
      contents: read
      pull-requests: write # (2)
    uses: projectpotos/actions/.github/workflows/test-pr-labels.yaml@v0.0.0
```

1. Deny all permissions at the workflow level as a secure baseline.
2. Required for the autolabeler to add labels to the pull request.

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `valid-labels` | Comma-separated labels of which at least one must be present | No | `bug, enhancement, feature, refactoring, major, deprecated, skip-changelog` |
| `invalid-labels` | Comma-separated labels none of which may be present | No | `help wanted, invalid, feedback-needed, incomplete` |
