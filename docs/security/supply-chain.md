# Security: Supply Chain

This page documents how the library implements the package **selection** and **integration**
controls from §4.1 and §4.2 of the
[ENISA Technical Advisory for Secure Use of Package Managers](https://www.enisa.europa.eu/publications/enisa-technical-advisory-for-secure-use-of-package-managers).

In this context "packages" means third-party GitHub Actions and reusable workflows referenced
in workflow files.

---

## Action Selection Criteria

*Implements ENISA TA §4.1.1 (Trusted Source), §4.1.4 (Maintainer Reputation),
§4.1.5 (Popularity & Maintenance), §4.1.6 (Secure Practices).*

Before adding any third-party action to a workflow, evaluate it against the following
criteria. All criteria should pass before the action is adopted. Document the evaluation in
the pull request or the associated
[New Workflow](https://github.com/projectpotos/actions/issues/new?template=new-workflow.yml)
/ [Update Workflow](https://github.com/projectpotos/actions/issues/new?template=update-workflow.yml)
issue.

### Trusted Source — §4.1.1

- [ ] The action is published under a **verified organisation account** (GitHub verified badge
      or a well-known project such as `actions/`, `github/`, `docker/`, `astral-sh/`,
      `zizmorcore/`) — or is maintained by the tool's own upstream.
- [ ] The action repository is the **canonical upstream** — not a fork, mirror, or re-publish.
- [ ] The action's repository URL is the same one referenced in the workflow (`uses:` field).

### Known Vulnerabilities — §4.1.2

- [ ] The action and its dependencies have been checked against the
      [GitHub Advisory Database](https://github.com/advisories) and
      [OSV.dev](https://osv.dev/) for known CVEs.
- [ ] No unmitigated HIGH or CRITICAL advisories exist for the action at the version being
      pinned.

### Signing & Integrity — §4.1.3

- [ ] The action commit being pinned is **reachable from a release tag** in the upstream
      repository (i.e. the SHA is not an arbitrary commit between releases).

### Maintainer Reputation — §4.1.4

- [ ] The maintaining organisation or individual has a **public track record** of security
      responsiveness (past CVE disclosures, security advisories, or responsible disclosure
      policy).
- [ ] The action has been in active use in major open-source projects (evidence via GitHub
      dependency graph or documented adoption).
- [ ] No recent ownership transfers or significant maintainer-base changes have occurred
      without community disclosure.

### Popularity & Maintenance — §4.1.5

- [ ] The action repository shows **recent commit activity** (within the last 6 months for
      actively used dependencies; within 12 months for stable/mature ones).
- [ ] The action has **open issue responsiveness**: maintainers respond to security issues
      within a reasonable time.
- [ ] The number of downstream users (GitHub Marketplace installs, stars, network dependents)
      indicates broad adoption — but note that popularity metrics can be gamed; use them as
      one signal, not the sole criterion.

### Secure Practices — §4.1.6

- [ ] The action's own workflow files follow security best practices (pinned dependencies,
      least-privilege permissions, no `pull_request_target` with `write` permissions on
      untrusted input).
- [ ] The action does not request unnecessary permissions in its own `action.yml`.
- [ ] The action does not run `curl | bash` or equivalent unbounded remote execution during
      setup.

---

## SHA Pinning

*Implements ENISA TA §4.2.3 (Integrity Enforcement) and §4.2.6 (Pinning Versions).*

Every third-party reference in this library — actions *and* reusable workflows — is pinned to
a **full commit SHA** with the version tag as an inline comment:

```yaml
uses: actions/checkout@9c091bb7c5b1e5c31b78d015a06e6cf5b0d29ee9 # v7.0.0
```

This provides two complementary controls in one line:

| What | How | ENISA Control |
|---|---|---|
| **Version pinning** | The SHA is immutable — unlike a version tag, it cannot be moved or deleted | §4.2.6 |
| **Integrity enforcement** | GitHub resolves the `uses:` reference by SHA; a different commit cannot be substituted | §4.2.3 |
| **Human readability** | The inline comment (`# v7.0.0`) shows which release the SHA corresponds to | — |

### Automation with Renovate

All dependencies are kept up-to-date automatically by [Renovate](https://docs.renovatebot.com/).

Configure Renovate in your repository by extending the shared
[projectpotos config](https://github.com/projectpotos/renovate-config):

```json title="renovate.json"
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>projectpotos/renovate-config"]
}
```

### Never Use Mutable References

These reference styles are **prohibited** in this library because they do not provide
integrity guarantees:

| Prohibited | Reason |
|---|---|
| `uses: actions/checkout@v7` | Mutable tag — can be moved silently |
| `uses: actions/checkout@main` | Branch reference — changes on every commit |
| `uses: actions/checkout@latest` | Alias — no integrity guarantee |

### Internal References Are Relative

**Consumers pin this repository by full commit SHA** as well. Internal references between
workflows in this repository are **relative** (`uses: ./.github/workflows/test-lint.yaml`),
which GitHub resolves at the same commit as the referencing workflow.

---

## Package Source Enforcement

*Implements ENISA TA §4.2.4 (Package Source Enforcement).*

Two independent controls restrict where workflow code can come from:

- **SHA pinning prevents source substitution**: if an attacker replaces a published action
  with malicious code at the same version tag, the SHA in our workflow will no longer match,
  and the run will fail. This is the GitHub Actions equivalent of enforcing a trusted
  registry URL.
- **Org-level `allowed_actions` allowlisting**: every potos repository has an explicit
  allowlist of runnable actions, managed centrally via OpenTofu. Same-org actions
  (`projectpotos/actions` itself) are always allowed by the GitHub org actions policy, so
  consumers need no entry for this library — but the third-party actions called *inside*
  these workflows do. An action outside the allowlist cannot run at all.

---

## Static Analysis

*Supports ENISA TA §4.1.6 (Secure Practices) and §4.2.2 (Vulnerability Checks).*

- [zizmor](https://docs.zizmor.sh/) runs with the **pedantic** persona on this repository
  (via the smoke tests) and must be clean. Suppressions live in `.github/zizmor.yml`, each
  with a documented reason.
- [actionlint](https://github.com/rhysd/actionlint) validates workflow syntax on every PR.

---

## Build Provenance

*Supports ENISA TA §4.1.3 (Signing & Integrity) for container images.*

`release-container.yaml` attaches a keyless
[build-provenance attestation](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations)
to every pushed image, verifiable with `gh attestation verify`. Downstream consumers of potos
images can verify that an image was built by the expected workflow from the expected commit.

---

## SBOM Gap — §4.2.1 (SBOM Creation)

This library provides workflow definitions only; it does not generate a Software Bill of
Materials for the workflows and their action dependencies. SBOM generation is therefore a
**gap** in this repository with respect to ENISA TA §4.2.1.

- **Status:** ❌ Gap
- **Mitigation:** Callers that build container images via `release-container.yaml` can add
  SBOM generation to their own release pipeline; build-provenance attestations are already
  attached to every pushed image.

!!! note "Future improvement"
    Automated SBOM generation for the action dependency graph (workflow references and
    transitive action dependencies) is tracked as a future improvement — see the
    [compliance matrix](index.md#controls-compliance-matrix).

---

## Installation Script Prevention

*ENISA TA §4.2.5 — Not Applicable.*

Traditional package managers (npm, pip) execute arbitrary scripts during installation
(`preinstall`, `postinstall`). GitHub Actions has no equivalent mechanism: an action is
either a Docker image, a JavaScript file, or a composite of shell steps — none of which
have an automatic install-script phase triggered by the `uses:` directive. This control is
therefore not applicable in the Actions context.

---

## Attribution

Controls in this page are derived from the
[ENISA Technical Advisory for Secure Use of Package Managers](https://www.enisa.europa.eu/publications/enisa-technical-advisory-for-secure-use-of-package-managers),
v1.1, March 2026. Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
