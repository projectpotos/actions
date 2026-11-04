# Ansible Collections: Draft Release

Drafts a collection release the way the
[Ansible community collections](https://github.com/ansible-collections) do:

1. [release-drafter](https://github.com/release-drafter/release-drafter) drafts the GitHub
   release and resolves the next version from the merged pull requests' labels (set by
   [PR Labels](../pr-labels.md)).
2. `antsibull-changelog release` opens that version in `changelogs/changelog.yaml`,
   consuming any fragments in `changelogs/fragments/`.
3. [antsichaut](https://github.com/ansible-community/antsichaut) adds every pull request
   merged since the latest release to `changelog.yaml`, categorised by label.
4. `antsibull-changelog generate` rewrites `CHANGELOG.rst`, `galaxy.yml` is bumped and a
   `chore: release vX.Y.Z` pull request is opened.

Merge that pull request, then publish the draft release; the `release: [published]` event
triggers [Release](release.md), which builds and publishes the collection.

Everything runs with a [GitHub App](https://docs.github.com/en/apps) installation token
rather than `GITHUB_TOKEN`, because a pull request created with `GITHUB_TOKEN` never
triggers workflows and the required checks would never report. The token is scoped to
`contents: write`, `pull-requests: write` and `metadata: read`.

## GitHub App

Create an app in the organisation with the repository permissions *Contents: read and
write*, *Pull requests: read and write* and *Metadata: read-only*, and install it on the
collection repositories. Store the app ID and private key either

- in OpenBao, as the fields `app_id` and `private_key` of one KV secret, with a JWT auth
  role bound to the calling repository (the `openbao-*` inputs), or
- as the `APP_ID` and `APP_PRIVATE_KEY` secrets of the repository or organisation.

antsichaut looks up the latest release through the GitHub releases API, so the repository
needs at least one GitHub release (pre-releases count) for the tag it last released from;
create one with `gh release create <tag>` if the tag was pushed by hand.

## Usage

Create a `.github/workflows/draft-release.yaml` file and a `.github/release-drafter.yml`
config (copy the one in this repository).

### "Secrets"

    ```yaml title=".github/workflows/draft-release.yaml"
    ---
    name: Draft release

    on:
      workflow_dispatch:

    permissions: {} # (1)

    jobs:
      draft-release:
        permissions:
          contents: read
          id-token: write # (2)
        uses: projectpotos/actions/.github/workflows/release-draft-ansible-collection.yaml@v0.0.0
        secrets:
          APP_ID: ${{ secrets.APP_ID }}
          APP_PRIVATE_KEY: ${{ secrets.APP_PRIVATE_KEY }}
    ```

### "OpenBao"

    ```yaml title=".github/workflows/draft-release.yaml"
    ---
    name: Draft release

    on:
      workflow_dispatch:

    permissions: {} # (1)

    jobs:
      draft-release:
        permissions:
          contents: read
          id-token: write # (2)
        uses: projectpotos/actions/.github/workflows/release-draft-ansible-collection.yaml@v0.0.0
        with:
          openbao-url: https://bao.example.com
          openbao-role: potos-readonly-ansible-collection-potos-base
          openbao-secret: potos-kv/data/github/apps/potos-release
    ```

1. Deny all permissions at the workflow level as a secure baseline.
2. Declared by the called job for the OIDC login to OpenBao; the caller must grant it even
   when the secrets variant is used. All writes go through the app token.

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `openbao-url` | OpenBao address. Leave empty to use the `APP_ID` and `APP_PRIVATE_KEY` secrets instead | No | `""` |
| `openbao-role` | JWT auth role to log in with | With `openbao-url` | `""` |
| `openbao-jwt-path` | Mount path of the JWT auth method | No | `jwt-github` |
| `openbao-audience` | Audience of the GitHub OIDC token | No | `https://github.com/<owner>` |
| `openbao-secret` | KV path of the app credentials, holding the fields `app_id` and `private_key` | With `openbao-url` | `""` |

## Secrets

| Secret | Description | Required |
|---|---|---|
| `APP_ID` | GitHub App ID | When `openbao-url` is empty |
| `APP_PRIVATE_KEY` | GitHub App private key (PEM) | When `openbao-url` is empty |
