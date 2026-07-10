## Summary

<!-- Describe the change and link to the related issue. -->

Closes #

## Type of Change

- [ ] 🐛 Bug fix — patch version bump
- [ ] ✨ New workflow — minor version bump
- [ ] 🔧 Update existing workflow — patch or minor version bump
- [ ] 🗑️ EOL / deprecate workflow
- [ ] 📖 Documentation only
- [ ] 🔒 Security improvement
- [ ] 🤖 Dependency / tooling update

## Checklist

- [ ] Workflow YAML is created or updated under `.github/workflows/`
- [ ] Documentation is created or updated under `docs/workflows/` (or `docs/actions/`)
- [ ] Permissions table in `docs/security/permissions.md` is updated
- [ ] `zensical.toml` nav is updated (required when a docs page is added or removed)
- [ ] `AGENTS.md` is updated (required when repository structure or conventions change)
- [ ] A smoke-test job exists (or the exception is documented in `AGENTS.md`)
- [ ] All caller examples include `permissions: {}` at the workflow level with explicit per-job permissions
- [ ] All third-party actions are pinned to a full commit SHA with version tag comment (e.g. `@abc1234 # v3`)
- [ ] Any new third-party actions have been evaluated against the [action selection criteria](https://projectpotos.github.io/actions/security/supply-chain/#action-selection-criteria)
- [ ] Consuming repositories' `allowed_actions` (tofu repo config) are updated for new third-party actions
- [ ] `uvx zizmor --persona pedantic .` and `uvx yamllint --strict .` are clean
- [ ] For breaking changes: migration guidance is included in the docs and the commit message contains `BREAKING CHANGE:`
- [ ] For EOL: deprecation notice is added to the workflow YAML and docs
