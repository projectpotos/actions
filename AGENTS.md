# Agent Instructions: projectpotos/actions

## Repository Purpose

This repository contains the [reusable GitHub Actions workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
and composite actions that Project Potos uses for CI/CD across all its projects. The goal is
that **all pipelines come from this repository**, so there is a single point for all external
dependencies. Workflows cover Ansible collection testing and Galaxy releases, container image
releases with build provenance, a generic lint suite, Zensical documentation deployment, and
semantic versioning.

The full documentation is published at [projectpotos.github.io/actions](https://projectpotos.github.io/actions/).

## Repository Structure

```
.github/workflows/          # Reusable workflow definitions (the actual product)
.github/ISSUE_TEMPLATE/     # Structured issue templates (bug, new-workflow, update, EOL)
.github/PULL_REQUEST_TEMPLATE.md  # PR checklist for all change types
.github/zizmor.yml          # zizmor suppressions (each with a documented reason)
setup-uv/                   # Composite action: checkout + install uv
all-green/                  # Composite action: required-check aggregation
docs/                       # Zensical source for the published documentation site
  security/                 # Security controls documentation
  workflows/                # One .md file per reusable workflow
  actions/                  # One .md file per composite action
tests/                      # Fixtures for the smoke tests
zensical.toml               # Zensical configuration (site name, nav)
```

## Conventions

### Workflow Files (`.github/workflows/`)

- **Naming**: `<verb>-<subject>.yaml` — e.g. `release-container.yaml`, `test-lint.yaml`.
  The file `semantic-release.yaml` is an explicit exception to mirror the upstream tool name.
- **Trigger**: All reusable workflows include `on: workflow_call`. `semantic-release.yaml` and
  `release-zensical.yaml` additionally trigger on pushes to `main` for this repository itself.
- **Permissions**: `permissions: {}` at the workflow level; declare `permissions:` on
  **every job** with the minimum set required and a comment for every write grant.
  See `docs/security/permissions.md` for the reference table.
- **Actions pinning**: Pin all third-party actions and reusable workflows to a full commit SHA
  with the version tag as a comment (e.g. `@abc123... # v3.1.0`). Dependabot keeps the pins
  current. Never use a mutable tag or branch for anything external.
- **Internal reuse**: Reference sibling reusable workflows **relatively**
  (`uses: ./.github/workflows/test-lint.yaml`) — GitHub resolves the reference at the same
  commit as the containing workflow, so consumers pinning one SHA get a consistent set.
  Composite actions can NOT be referenced relatively from reusable workflows (step-level `./`
  paths resolve against the consumer's checkout); shared job logic therefore lives in
  reusable workflows, not composites.
- **Inputs in shell**: Treat all `inputs.*` values as untrusted. Pass them through `env:`
  variables instead of interpolating `${{ }}` directly into `run:` blocks.
- **Side-effect gating**: Guard every push/publish/deploy step with
  `if: github.event_name != 'pull_request'` (or a boolean input like `deploy`/`dry`) so the
  workflows stay smoke-testable and fork PRs cannot trigger side effects.

### Composite Actions (`setup-uv/`, `all-green/`)

- One directory per action, containing a single `action.yml`.
- For step-level use inside consumer-specific jobs only (e.g. a pytest matrix job that runs
  `setup-uv` and then a custom command). Anything that is a complete job belongs in a
  reusable workflow instead.

### Conventional Commits and Versioning

Commit messages follow the [Conventional Commits](https://www.conventionalcommits.org/)
specification. Releases are automated by go-semantic-release on every push to `main`
(`semantic-release.yaml`):

- `feat:` — new workflow or new input → **minor** version bump
- `fix:` — bug fix in a workflow → **patch** bump
- `docs:`, `ci:`, `chore:` — no release unless combined with the above
- `BREAKING CHANGE:` footer on any type → **major** bump

All work happens on feature branches. Open a PR to `main`; do not manually create version tags.

## Workflows Provided

| File | Purpose |
|---|---|
| `release-ansible-collection.yaml` | Publish Galaxy release |
| `release-container.yaml` | Build, push, and attest container images (GHCR by default) |
| `release-zensical.yaml` | Build and deploy Zensical docs to GitHub Pages (also deploys this repo's docs) |
| `semantic-release.yaml` | Automate releases with go-semantic-release (also releases this repo) |
| `test-ansible-collection.yaml` | Full Ansible collection CI (upstream ansible suite + lint + molecule) |
| `test-lint.yaml` | Lint suite: yamllint, actionlint, zizmor always; ruff, hadolint, shellcheck, shellcheck-jinja opt-in |
| `smoke-test.yaml` | This repo's CI: smoke tests for the test workflows (required check: `all_green`) |
| `smoke-test-release.yaml` | This repo's CI: smoke tests for the release workflows |

Composite actions: `setup-uv` (checkout + uv), `all-green` (required-check aggregation).

## Making Changes

### Adding a new reusable workflow

1. Create `.github/workflows/<verb>-<subject>.yaml` with `on: workflow_call`.
2. Add a corresponding documentation file: grouped workflows go in
   `docs/workflows/<category>/<name>.md`; standalone workflows go directly in
   `docs/workflows/<name>.md`.
3. Register the new page in `zensical.toml` under `nav`.
4. Update `docs/security/permissions.md` with the new workflow's required permissions.
5. Add a smoke-test job to `smoke-test.yaml` (test workflows) or `smoke-test-release.yaml`
   (release workflows), or document below why it cannot be smoke-tested.
6. Update this file's workflow table and the caller's `allowed_actions` in the tofu repo
   config (`tofu-config-potos-github/config/repos.yml`) if new third-party actions are used.

Smoke-test exceptions:

- `test-ansible-collection.yaml` cannot be smoke-tested in this repository: the upstream
  `ansible/ansible-content-actions` workflows it calls (changelog, build-import, ansible-lint,
  sanity, unit) operate on the caller's repository root and take no path input, so they
  cannot be pointed at the `tests/ansible/` fixture. The workflow is exercised by every
  collection repository's CI instead. (`release-ansible-collection.yaml` *is* smoke-tested
  against the fixture)

### Updating an existing workflow

1. Edit the workflow YAML.
2. Keep the documentation in sync (inputs table, permissions table, usage example).

### Deprecating or removing a workflow

1. Open an `eol-workflow.yml` issue to announce intent and gather feedback.
2. Add a deprecation notice to the workflow YAML (as a comment) and to the docs page.
3. After the deprecation window (at least one minor release), remove the workflow file, its
   documentation page, its `zensical.toml` nav entry, and its row in
   `docs/security/permissions.md`.

### Pull Requests

All changes go through a pull request. `.github/PULL_REQUEST_TEMPLATE.md` contains the
checklist; key items are documentation sync, permissions table updates, SHA pinning, and
smoke-testability.

## Linting and Testing

### CI smoke tests

Reusable workflows are smoke-tested on each PR to `main` by calling the **local**
(current-branch) version via `uses: ./.github/workflows/...`:

| Smoke test job | File | Workflow under test | Test strategy |
|---|---|---|---|
| `smoke-test-lint` | `smoke-test.yaml` | `test-lint.yaml` | Lint this repo (`yamllint-mode: standalone`, `zizmor-persona: pedantic`); shellcheck-jinja against the `tests/shellcheck-jinja/` fixture |
| `smoke-semantic-release` | `smoke-test.yaml` | `semantic-release.yaml` | `dry: true` — compute next version, create nothing |
| `smoke-release-container` | `smoke-test-release.yaml` | `release-container.yaml` | Build `tests/container/Dockerfile` (`FROM scratch`); push/attest gated on non-PR events |
| `smoke-release-zensical` | `smoke-test-release.yaml` | `release-zensical.yaml` | Build this repo's docs; `deploy: false` |

The `all_green` job in `smoke-test.yaml` is the repository's single required status check
(see `required_status_checks` in the tofu repo config).

Workflows **not** smoke-tested and why:

- `test-ansible-collection.yaml` — the upstream `ansible/ansible-content-actions` workflows
  operate on the repository root and provide no `path` input, so they cannot be pointed at a
  fixture. Validated in the consuming collection repositories.
- `release-ansible-collection.yaml` — requires a locked uv project and `galaxy.yml` at the
  repository root, plus a matching version tag and the `release` environment. Validated in
  the consuming collection repositories.

### Making workflows smoke-testable

- **Required secrets**: use `required: false` where possible, or gate the consuming step.
- **Side effects**: add boolean inputs (`deploy`, `dry`, ...) or gate on
  `github.event_name != 'pull_request'` so smoke tests cannot publish anything.
- **Working directory**: prefer `context`/`path` inputs so smoke tests can target fixtures
  under `tests/`.
- **Permissions**: the calling job must grant **every permission the called workflow's jobs
  declare — even jobs that are skipped** by an `if:` or input; GitHub validates this at
  workflow-call time. Comment every elevated grant with why it is safe (side effect gated
  off). Put smoke jobs whose called workflow declares write permissions into
  `smoke-test-release.yaml` where those grants are documented.

### Local linting

```bash
uvx yamllint --strict --format github .
uvx zizmor --persona pedantic .
```

Both must be clean before merging; the smoke tests enforce the same checks in CI.
Suppressions go into `.github/zizmor.yml` with a comment explaining the reason.

### Docs preview

```bash
pip install zensical
zensical serve
```

## Related Repositories

- [tofu-config-potos-github](https://github.com/projectpotos/repo-config-tofu) — manages this
  repo's settings, `allowed_actions`, and required status checks via OpenTofu
- Consumer repos: `ansible-collection-potos.base`, `potos-iso-builder`,
  `docker-fedora44-ansible`, `potos-iso-builder-template`
