# Contributing

## Workflow Lifecycle

### Proposing a new workflow

Open a [New Workflow issue](https://github.com/projectpotos/actions/issues/new/choose)
using the structured template. A workflow belongs in this library when it is (or will be)
useful to more than one potos repository — the goal is that **all pipelines come from this
repository**.

### Adding a workflow

1. Create `.github/workflows/<verb>-<subject>.yaml` with `on: workflow_call`
   (`test-`, `release-`, or `smoke-test-` prefix).
2. Document it under `docs/workflows/` (see the page structure in `docs/AGENTS.md`) and
   register the page in `zensical.toml` under `nav`.
3. Add the workflow to the [permissions reference](security/permissions.md).
4. Add a smoke-test job to `smoke-test.yaml` or `smoke-test-release.yaml`, or document in
   `AGENTS.md` why it cannot be smoke-tested.
5. Evaluate new third-party actions against the
   [action selection criteria](security/supply-chain.md#action-selection-criteria) and pin
   them by full commit SHA.
6. If consuming repositories need new `allowed_actions` entries, update the OpenTofu repo
   config accordingly.

### Updating a workflow

Keep the documentation in sync in the same PR: inputs table, permissions table, and usage
example.

### Deprecating a workflow

1. Open an EOL issue to announce intent and gather feedback.
2. Add a deprecation notice to the workflow YAML (as a comment) and to its docs page.
3. After at least one minor release, remove the workflow, its docs page, its nav entry,
   and its permissions table row.

## Conventional Commits and Releases

Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/).
[go-semantic-release](https://go-semantic-release.xyz/) creates a release on every push to
`main`:

| Commit type | Version bump |
|---|---|
| `feat:` | minor |
| `fix:` | patch |
| `BREAKING CHANGE:` footer | major |
| `docs:`, `ci:`, `chore:` | none |

All work happens on feature branches and is merged via pull request. Do not create version
tags manually.

## Local Checks

```bash
uvx yamllint --strict --format github .
uvx zizmor --persona pedantic .
```

Both must be clean; the smoke tests enforce the same checks in CI. To preview the
documentation site:

```bash
pip install zensical
zensical serve
```
