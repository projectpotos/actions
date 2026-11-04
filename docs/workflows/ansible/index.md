# Ansible Collections

Workflows for repositories that contain an Ansible collection:

| Workflow | Description |
|---|---|
| [Test](test.md) | Full collection CI: upstream ansible suite, lint suite, changelog lint, molecule |
| [Draft Release](draft-release.md) | Draft the release: version from the PR labels, changelog from the PRs and fragments, release PR |
| [Release](release.md) | Build and publish the collection to Ansible Galaxy |

The release flow is the one the Ansible community collections use: pull request titles follow
Conventional Commits, [PR Labels](../pr-labels.md) turns them into labels, Draft Release
turns the labels into the version and the changelog, and publishing the draft triggers
Release.
