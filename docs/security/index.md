# Security

This library is the single point for all external CI/CD dependencies of Project Potos.
Concentrating the pipelines here is itself the primary supply-chain control: third-party
actions are selected, pinned, audited, and updated in exactly one place.

## Controls

| Control | Implementation |
|---|---|
| Full SHA pinning of everything external | Every `uses:` reference to a third-party action or reusable workflow is pinned to a full commit SHA with the version tag as a comment; see [Supply Chain](supply-chain.md) |
| Version consistency for internal reuse | Sibling workflows are referenced relatively and resolve at the same commit as the consumer's pin |
| Least-privilege permissions | `permissions: {}` at the workflow level, minimum per-job grants, every write grant commented; see [Permissions](permissions.md) |
| Static analysis | zizmor (pedantic persona) and actionlint run on every PR via the smoke tests; suppressions live in `.github/zizmor.yml` with documented reasons |
| Injection hardening | All `inputs.*` values reach `run:` blocks via `env:` variables, never by direct interpolation |
| Side-effect gating | Push/publish/deploy/attest steps are gated on non-PR events or explicit boolean inputs |
| Automated dependency updates | Dependabot updates all SHA pins daily with a 7-day cooldown |
| Build provenance | Container images get a keyless build-provenance attestation pushed to the registry |
| Org-level enforcement | `allowed_actions` per repository and required status checks are managed centrally via OpenTofu |
