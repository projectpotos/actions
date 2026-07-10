# Security: Supply Chain

## Pinning Policy

- **Everything external is pinned to a full commit SHA** with the version tag as a trailing
  comment: `uses: actions/checkout@9c091bb... # v7.0.0`. This applies to third-party
  actions *and* third-party reusable workflows (including the upstream
  `ansible/ansible-content-actions` suite).
- **Dependabot** updates all pins daily (SHA and version comment together), with a 7-day
  cooldown against freshly-released versions.
- **Consumers pin this repository by full commit SHA** as well. Internal references between
  workflows in this repository are **relative** (`uses: ./.github/workflows/test-lint.yaml`),
  which GitHub resolves at the same commit as the referencing workflow — one consumer pin
  yields one consistent version of the whole library, with no floating internal refs.
- Same-org actions are always allowed by the GitHub org actions policy, so
  `projectpotos/actions` itself needs no `allowed_actions` entry in consuming repositories —
  but the third-party actions called *inside* these workflows do. `allowed_actions` is
  managed centrally via OpenTofu.

## Action Selection Criteria

Before adding a new third-party action, evaluate it:

1. Prefer GitHub first-party (`actions/*`, `github/*`) or verified-creator actions.
2. Prefer actions maintained by the tool's own upstream (e.g. `astral-sh/setup-uv`,
   `docker/*`, `zizmorcore/zizmor-action`).
3. Check for known vulnerabilities (GitHub Advisory Database, OSV.dev) and general
   maintenance health (recent releases, responsive issues).
4. Review what the action does with the token and whether it needs any write permission.
5. Pin to a full commit SHA from day one and document the evaluation in the PR.

## Static Analysis

- [zizmor](https://docs.zizmor.sh/) runs with the **pedantic** persona on this repository
  (via the smoke tests) and must be clean. Suppressions live in `.github/zizmor.yml`, each
  with a documented reason.
- [actionlint](https://github.com/rhysd/actionlint) validates workflow syntax on every PR.
- All `inputs.*` values are passed to shell steps via `env:` variables — never interpolated
  directly into `run:` blocks — to prevent template injection.

## Build Provenance

`release-container.yaml` attaches a keyless
[build-provenance attestation](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations)
to every pushed image, verifiable with `gh attestation verify`.
