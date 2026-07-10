# Container Images

Workflows for repositories that build container images:

| Workflow | Description |
|---|---|
| [Release](release.md) | Build, push (GHCR by default), and attest container images |

Used by `potos-iso-builder` (semver-tagged releases) and `docker-fedora44-ansible`
(rolling `latest` with a weekly rebuild cron in the caller).
