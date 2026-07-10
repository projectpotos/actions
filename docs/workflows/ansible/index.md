# Ansible Collections

Workflows for repositories that contain an Ansible collection (there will be more than one —
the CI and release pipelines are fully shared):

| Workflow | Description |
|---|---|
| [Test](test.md) | Full collection CI: upstream ansible suite, lint suite, changelog lint, molecule |
| [Release](release.md) | Version, build, and publish the collection to Ansible Galaxy |

Both workflows expect a collection repository with `galaxy.yml`, `changelogs/`, and a locked
uv project (`pyproject.toml` + `uv.lock`) providing `antsibull-changelog`, `yamllint`,
`ruff`, and `molecule`.
