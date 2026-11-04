# PR Labels

Labels a pull request from its [Conventional Commits](https://www.conventionalcommits.org/)
title with the [release-drafter](https://github.com/release-drafter/release-drafter)
autolabeler, then runs release-drafter's `check-pr` action, which fails unless the title or
the labels match one of the categories in `.github/release-drafter.yml`. The labels are
what the collection changelog ([Draft Release](ansible/draft-release.md)) is generated from.

| Title prefix | Label | Version bump |
|---|---|---|
| `feat:`, `perf:` | `enhancement` | minor |
| `fix:` | `bug` | patch |
| `refactor:`, `style:` | `refactoring` | patch |
| `<type>!:` | `major` | major |
| `chore:`, `ci:`, `test:`, `build:`, `docs:` | `skip-changelog` | none, hidden from the notes |

The categories match on the title *or* the label, so the check passes in the same run that
adds the labels, and a hand-labelled pull request passes without a conventional title. Pull
requests from forks run with a read-only token, so the autolabeler cannot label them; a
maintainer adds the label by hand. With `sync-labels: true` an edited title replaces the
labels the autolabeler manages.

## Usage

Create a `.github/workflows/pr-labels.yaml` file and a `.github/release-drafter.yml`
config (copy the one in this repository).

```yaml title=".github/workflows/pr-labels.yaml"
---
name: PR labels

on:
  pull_request:
    types: [opened, reopened, synchronize, edited, labeled, unlabeled]

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

None.
