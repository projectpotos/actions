# Workflow Analysis

Analysis of all GitHub Actions workflows in the tofu-managed potos repositories (July 2026),
the duplication between them, and how this repository deduplicates them. This is the design
background for the library; the current state of the library itself is documented in
[Workflows](workflows/index.md).

## Inventory (before migration)

| Repository | Workflow | Jobs |
|---|---|---|
| ansible-collection-potos.base | `tests.yml` | changelog, build-import, ansible-lint, sanity, unit-galaxy, unit-source (all via `ansible/ansible-content-actions` + `ansible-network/github_actions` reusable workflows), ruff, yamllint, changelog-lint, molecule (6-scenario matrix), all_green |
| ansible-collection-potos.base | `release.yml` | Galaxy release: version resolution, galaxy.yml bump, antsibull-changelog, collection build + publish, commit-back, artifact upload |
| potos-iso-builder | `tests.yml` | pytest (3-version matrix), ruff, yamllint, hadolint, shellcheck, actionlint, zizmor, all_green |
| potos-iso-builder | `docker-publish.yml` | buildx build → GHCR push (semver + edge + sha tags) → provenance attestation |
| docker-fedora44-ansible | `lint.yml` | yamllint, hadolint, actionlint, zizmor, all_green |
| docker-fedora44-ansible | `docker-publish.yml` | QEMU + buildx build → GHCR push (`latest` only, weekly cron rebuild) → provenance attestation |
| potos-iso-builder-template | — | no workflows yet (only dependabot config) |

Out of scope (not tofu-managed, noted for later): `docs` (zensical → GitHub Pages, pinned,
healthy) and `projectpotos.github.io` (legacy Node/vuepress deploy: unpinned tag refs, no
`permissions:` block, `persist-credentials` left at default — worth migrating or archiving).

## Duplication found

1. **checkout + setup-uv + run a tool** — 9 jobs across 3 repos (ruff ×2, yamllint ×3,
   changelog-lint, molecule, pytest). Identical boilerplate, only the final command differs.
2. **Pinned third-party wrapper jobs** — hadolint ×2, actionlint ×2, zizmor ×2,
   shellcheck ×1. The value of centralizing is one place to manage the SHA pins instead of
   N repos drifting (they had already drifted: checkout v5.0.1 vs v7.0.0, setup-uv v7.6.0 vs
   v8.2.0, actionlint v2.1.2 vs v2.2.0, zizmor-action v0.5.6 vs v0.5.7, and the whole docker
   action family one major apart). The ansible-content-actions workflows were consumed
   `@main` — now hash-pinned here (v1.1.0).
3. **`all_green` aggregate job** — 3 near-identical copies. The ansible-collection copy
   interpolated `${{ toJSON(needs.*.result) }}` directly into the run block (an
   injection-unsafe pattern zizmor warns about); the shared `all-green` composite passes it
   via an env var.
4. **docker-publish job** — 2 near-identical 90-line jobs (login → metadata → build-push →
   attest). Differences are only tag strategy, platforms/QEMU, and OCI labels — all
   parametrized in `release-container.yaml`.
5. **Ansible collection CI + release** — currently one repo, but more collection
   repositories are planned, so the whole tests.yml job set and the Galaxy release workflow
   are shared (`test-ansible-collection.yaml`, `release-ansible-collection.yaml`).

## Architecture

- **Reusable workflows** (`.github/workflows/`) are the primary unit; the file naming
  follows `<verb>-<subject>.yaml`.
- **Internal reuse via relative references**: nested reusable workflow calls
  (`uses: ./.github/workflows/test-lint.yaml`) resolve at the same commit as the workflow
  containing the reference, so one consumer pin yields one consistent version of
  everything. Composite actions can NOT be referenced this way (step-level relative paths
  resolve against the consumer's checkout), which is why per-tool lint composites were
  dropped in favour of the nested lint workflow.
- **Composite actions** (`setup-uv`, `all-green`) remain for consumer-side,
  repo-specific jobs. Consumers reference them remotely by SHA; GitHub materializes them
  outside the workspace, so they don't pollute lint scans.
- Repo-specific by design: release triggers and cron schedules (must live in the caller),
  and the caller's `all_green` aggregation.

## Expected consumer migration

| Repo / workflow | Change |
|---|---|
| ansible-collection-potos.base `tests.yml` | → single call to `test-ansible-collection.yaml` (molecule scenarios as input) + `all-green`. Adds actionlint + zizmor coverage the repo didn't have; fixes the unsafe toJSON interpolation. |
| ansible-collection-potos.base `release.yml` | → `release-ansible-collection.yaml` (keeps `release` environment + `ANSIBLE_GALAXY_API_KEY` secret in the repo) |
| potos-iso-builder `tests.yml` | lint jobs → `test-lint.yaml` (`ruff: true, ruff-group: lint, hadolint: true, shellcheck: true`); pytest matrix keeps the `setup-uv` composite; `all-green` composite |
| potos-iso-builder `docker-publish.yml` | → `release-container.yaml` (semver/edge/sha tags) |
| docker-fedora44-ansible `lint.yml` | → `test-lint.yaml` (`yamllint-mode: standalone`, `hadolint: true`) + `all-green` |
| docker-fedora44-ansible `docker-publish.yml` | → `release-container.yaml` (`setup-qemu: true`, `latest` tag; cron trigger stays in caller) |
| potos-iso-builder-template | can gain a minimal lint workflow via `test-lint.yaml` once desired |
| future collection repos | copy the two ~20-line caller workflows from the docs |

`allowed_actions` in the tofu repo config: same-org references need no entry; the
third-party actions used *inside* the reusable workflows must be allowed in each consuming
repository. After migration, the collection repos additionally need
`raven-actions/actionlint@*` and `zizmorcore/zizmor-action@*` (new coverage); entries for
actions no longer used directly can be pruned.
