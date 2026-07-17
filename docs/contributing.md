# Contributing

This guide explains how to report issues, propose changes, and submit pull requests for the
reusable workflows and composite actions in this library.

## Security Review

All changes that introduce or update a third-party action, change permissions, or affect
the security posture of the library must be reviewed against the controls documented in
the [Security](security/index.md) section.

### When the security docs apply

| Change type | Required review |
|---|---|
| New third-party action | Full [action selection criteria](security/supply-chain.md#action-selection-criteria) evaluation; document in issue or PR |
| Updated SHA pin | Confirm the new SHA is reachable from a release tag |
| Permission change | Verify against the [Permissions](security/permissions.md) reference; justify any increase |
| Vulnerability fix | Follow the [mitigation steps](security/vulnerability-management.md#mitigation-steps) and [notification process](security/vulnerability-management.md#notification) |
| Any other change | Work through the security items in the PR checklist |

### Quarterly ENISA alignment review

Once per quarter, a maintainer should open an
[Update Workflow](https://github.com/projectpotos/actions/issues/new?template=update-workflow.yml)
issue with type **Security improvement** to:

1. Check the [EUVD](https://euvd.enisa.europa.eu/) and [OSV.dev](https://osv.dev/) for new
   advisories affecting actions used in this library.
2. Verify all open Dependabot alerts have been reviewed and are within the response time
   thresholds in [Vulnerability Management](security/vulnerability-management.md#prioritisation).
3. Update the [compliance matrix](security/index.md#controls-compliance-matrix) if any
   control status has changed.
4. Review and triage any `security`-labelled open issues.

## Reporting Issues

Use the structured issue templates to open requests:

| Template | When to use |
|---|---|
| [Bug Report](https://github.com/projectpotos/actions/issues/new?template=bug-report.yml) | A workflow behaves incorrectly or fails unexpectedly |
| [New Workflow](https://github.com/projectpotos/actions/issues/new?template=new-workflow.yml) | You want to add a new reusable workflow to the library |
| [Update Workflow](https://github.com/projectpotos/actions/issues/new?template=update-workflow.yml) | A workflow needs a new input, a tool version bump, or a behavior change |
| [EOL / Deprecate](https://github.com/projectpotos/actions/issues/new?template=eol-workflow.yml) | A workflow should be deprecated or removed |

## Workflow Lifecycle

A workflow belongs in this library when it is (or will be) useful to more than one potos
repository — the goal is that **all pipelines come from this repository**.

### Adding a workflow

1. Open a [New Workflow](https://github.com/projectpotos/actions/issues/new?template=new-workflow.yml)
   issue to discuss the proposal before writing code.
2. Create `.github/workflows/<verb>-<subject>.yaml` with `on: workflow_call`
   (`test-`, `release-`, or `smoke-test-` prefix).
3. Document it under `docs/workflows/` (see the page structure in `docs/AGENTS.md`) and
   register the page in `zensical.toml` under `nav`.
4. Add the workflow to the [permissions reference](security/permissions.md).
5. Add a smoke-test job to `smoke-test.yaml` or `smoke-test-release.yaml`, or document why it cannot be smoke-tested.
6. Evaluate new third-party actions against the
   [action selection criteria](security/supply-chain.md#action-selection-criteria) and pin
   them by full commit SHA.
7. If consuming repositories need new `allowed_actions` entries, update the OpenTofu repo
   config accordingly.
8. Update `AGENTS.md` if the repository structure or conventions change, then submit a
   pull request.

### Updating a workflow

1. Open an [Update Workflow](https://github.com/projectpotos/actions/issues/new?template=update-workflow.yml)
   issue for non-trivial changes to discuss the approach first.
2. Edit the workflow YAML under `.github/workflows/`.
3. Keep the documentation in sync in the same PR: inputs table, permissions table, and
   usage example.
4. For breaking changes, include a migration path in the docs and add a `BREAKING CHANGE:`
   footer to the commit message to trigger a major version bump.

### Deprecating or removing a workflow

1. Open an [EOL / Deprecate](https://github.com/projectpotos/actions/issues/new?template=eol-workflow.yml)
   issue to announce intent and gather feedback from users.
2. Add a deprecation notice to the workflow YAML (as a comment) and to its docs page.
3. After the deprecation window (at least one minor release), remove the workflow file, its
   documentation page, its nav entry in `zensical.toml`, and its row in
   `docs/security/permissions.md`.

## Pull Requests

All work happens on feature branches and is merged via pull request to `main`. The PR
template prompts you through the required checklist: documentation and permissions table in
sync, smoke-test coverage, `permissions: {}` in caller examples, SHA-pinned third-party
actions evaluated against the
[selection criteria](security/supply-chain.md#action-selection-criteria), and
`allowed_actions` updates in the OpenTofu repo config.

Releases are automated: [go-semantic-release](https://go-semantic-release.xyz/) creates one
on every push to `main`, so do not create version tags manually. Commit messages follow
[Conventional Commits](https://www.conventionalcommits.org/):

| Commit type | Version bump |
|---|---|
| `feat:` | minor |
| `fix:` | patch |
| `BREAKING CHANGE:` footer | major |
| `docs:`, `ci:`, `chore:` | none |

## Downstream Projects

Projects that adopt these workflows can copy and adapt `.github/ISSUE_TEMPLATE/` and
`.github/PULL_REQUEST_TEMPLATE.md` from this repository to standardise their own CI/CD
configuration changes, and should add the
[Renovate configuration](getting-started.md#usage) to keep workflow SHA pins up to date
automatically.