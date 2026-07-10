# Agent Instructions: .github/workflows/

## Purpose

This directory contains all reusable GitHub Actions workflows for Project Potos. Each file is
a self-contained, callable workflow invoked via
`uses: projectpotos/actions/.github/workflows/<file>@<sha>`.

## File Naming

```
<verb>-<subject>.yaml
```

Allowed verbs:

| Verb | When to use |
|---|---|
| `release-` | Builds an artifact and publishes/deploys it |
| `test-` | Runs linting, unit tests, or other quality checks |
| `smoke-test-` | This repository's own CI (calls the local workflows) |

`semantic-release.yaml` is an explicit exception — it mirrors the upstream tool name.

## Workflow Skeleton

```yaml
---
name: <Human Readable Name>

on:
  workflow_call:
    inputs:
      <input-name>:
        description: <what this input does>
        type: <string|boolean|number>
        required: <true|false>
        default: <default value>   # omit when required: true
    secrets:
      <SECRET_NAME>:
        description: <what this secret is used for>
        required: <true|false>

permissions: {}

jobs:
  <job-name>:
    name: <human readable job name>
    runs-on: ubuntu-latest
    timeout-minutes: <n>
    permissions:
      <permission>: <read|write> # comment explaining every write grant
    steps:
      - name: Checkout
        uses: actions/checkout@<sha> # vN.N.N
        with:
          persist-credentials: false
      # ...
```

## Branching and Versioning

- All work happens on feature branches; merge to `main` via pull requests.
- go-semantic-release automatically creates a new release on every push to `main`.
- Conventional commit types determine the version bump:
  - `feat:` → minor bump
  - `fix:` → patch bump
  - `BREAKING CHANGE:` footer → major bump
  - `docs:`, `ci:`, `chore:` → no release by themselves
- Do not manually create or push version tags.

## Permissions Rules

- Set `permissions: {}` at the workflow level; grant per job only.
- Start from zero and add only what the job strictly requires.
- Comment every write grant with why it is needed and how it is gated.
- When adding or removing a permission, update `docs/security/permissions.md` in the same
  commit.

## Actions Pinning

- Pin **everything external** (actions and reusable workflows) to a full commit SHA with the
  version tag as a comment: `uses: owner/repo@<sha> # vN.N.N`.
- Dependabot updates the SHA and the comment together; never hand-edit only one of them.
- Reference **sibling workflows relatively** (`uses: ./.github/workflows/test-lint.yaml`) —
  no ref needed; GitHub resolves them at the same commit as the containing workflow.
- Composite actions in this repository cannot be referenced relatively from reusable
  workflows (step-level `./` paths resolve against the consumer's checkout). Shared job
  logic belongs in a reusable workflow.

## Security Practices

- Treat all `inputs.*` values as untrusted strings when constructing shell commands.
  Pass values through environment variables (`env:`) rather than interpolating `${{ }}`
  directly into `run:` blocks.
- Use `if: github.event_name != 'pull_request'` guards (or boolean inputs like `deploy`,
  `dry`) before any step that pushes, signs, publishes, or deploys.
- `persist-credentials: false` on every checkout
- Run `uvx zizmor --persona pedantic .` — it must be clean (suppressions only via a
  `.github/zizmor.yml` with a documented reason; currently none are needed).

## Adding a New Workflow

1. Create `.github/workflows/<verb>-<subject>.yaml` following the skeleton above.
2. Add `docs/workflows/<category>/<name>.md` documenting it (see `docs/AGENTS.md`).
3. Register the page in `zensical.toml` under `nav`.
4. Add the workflow to the permissions reference table in `docs/security/permissions.md`.
5. Add a smoke-test job (see the root `AGENTS.md`, section "CI smoke tests").
6. If new third-party actions are introduced, add them to the consumers' `allowed_actions`
   in the tofu repo config (`tofu-config-potos-github/config/repos.yml`).
