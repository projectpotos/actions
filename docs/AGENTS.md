# Agent Instructions: docs/

## Purpose

This directory contains the [Zensical](https://zensical.org/) source for the documentation
site published at [projectpotos.github.io/actions](https://projectpotos.github.io/actions/).

The documentation explains how to use every reusable workflow in `.github/workflows/` and
every composite action, with copy-pasteable YAML snippets, input tables, and permission
references.

## Directory Layout

```
docs/
  index.md              # Home page (hero + feature grid, styled via stylesheets/extra.css)
  getting-started.md    # Quickstart and security baseline instructions
  contributing.md       # Workflow lifecycle, security review, and process guide
  security/
    index.md            # ENISA TA threat model and controls compliance matrix
    permissions.md      # Reference table: workflow → required permissions
    supply-chain.md     # ENISA §4.1/§4.2: action selection, SHA pinning, source enforcement
    vulnerability-management.md  # ENISA §4.3/§4.4: monitoring, assessment, mitigation
  workflows/
    index.md            # Workflow category overview table
    ansible/
      index.md          # Ansible category landing page
      test.md           # test-ansible-collection.yaml docs
      release.md        # release-ansible-collection.yaml docs
    container/
      index.md          # Container category landing page
      release.md        # release-container.yaml docs
    lint.md             # test-lint.yaml docs
    zensical.md         # release-zensical.yaml docs
    semantic-release.md # semantic-release.yaml docs
  actions/
    index.md            # Composite actions landing page
    setup-uv.md         # setup-uv composite action docs
    all-green.md        # all-green composite action docs
```

## Page Structure for Each Workflow

Every workflow documentation page must contain these sections in order:

1. **Title** — `# <Category>: <Action>` and a one-sentence description
2. **`## Usage`** — copy-pasteable caller workflow with `permissions: {}` at the workflow
   level, per-job permission grants, and `uses: projectpotos/actions/...@v0.0.0`
   (never `@main`/`@latest`)
3. **`## Inputs`** — table with Input / Description / Required / Default columns
   (mark required inputs as `**Yes**`)
4. **`## Secrets`** — table in the same format (only when the workflow accepts secrets)

Composite action pages under `docs/actions/` follow the same structure, with the usage
example showing the action as a step inside a job.

## Key Conventions

- **Always** include `permissions: {}` at the top level of every usage example, followed by
  per-job permission grants using only the minimum required (matching
  `security/permissions.md`).
- **Version placeholder**: Examples reference `@v0.0.0`. This reminds readers to substitute
  the actual current release (pin the full commit SHA with the tag as a comment). Do not use
  `@main` or `@latest`.
- Numbered explanations for individual YAML keys go as a list immediately after the fenced
  code block.
- Update `zensical.toml` `nav` whenever a documentation page is added or removed.

## Local Preview

```bash
pip install zensical
zensical serve
```

Open `http://127.0.0.1:8000` to preview changes before committing.
