# homelab-toolbox-auth

GitHub Composite Action that configures authenticated Git access to the private helax-homelab/homelab-toolbox repository.

The action creates a GitHub App installation token scoped to the toolbox repository and uses [helax-homelab-actions/git-url-auth](https://github.com/helax-homelab-actions/git-url-auth) to configure Git authentication.

Both the Git URL rewrite and the GitHub App token are automatically cleaned up during the action's post-job cleanup.

## Usage

```yaml
- name: Configure homelab toolbox access
  uses: helax-homelab-actions/homelab-toolbox-auth@v1
  with:
    client-id: ${{ secrets.HOMELAB_TOOLBOX_APP_CLIENT_ID }}
    private-key: ${{ secrets.HOMELAB_TOOLBOX_APP_PRIVATE_KEY }}
```

Once configured, Git dependencies referencing the toolbox through its SSH URL can be fetched normally:

```text
ssh://git@github.com/helax-homelab/homelab-toolbox.git
```

## Inputs

| Input         | Required | Description            |
| ------------- | -------- | ---------------------- |
| `client-id`   | Yes      | GitHub App client ID   |
| `private-key` | Yes      | GitHub App private key |

The GitHub App must have **Contents: read** permission on the helax-homelab/homelab-toolbox repository.

## Outputs

This action does not produce any outputs. The GitHub App token is created internally and passed to git-url-auth for Git configuration.

## How it works

The action is a composition of two reusable actions:

1. [actions/create-github-app-token](https://github.com/actions/create-github-app-token) creates a GitHub App installation token scoped to:
   - **Owner**: helax-homelab
   - **Repository**: homelab-toolbox
   - **Permission**: contents: read

2. [helax-homelab-actions/git-url-auth](https://github.com/helax-homelab-actions/git-url-auth) configures Git to rewrite the toolbox repository's SSH URL to an authenticated HTTPS URL.

The resulting Git configuration only applies to the toolbox repository.

## Cleanup

Each action is responsible for cleaning up the state it creates.

`git-url-auth` removes the Git URL rewrite during its post-job cleanup.

`actions/create-github-app-token` revokes the GitHub App installation token during its post-job cleanup.

This is particularly important on self-hosted runners, where the same runner may execute multiple jobs over its lifetime.

## Repository

This action authenticates access specifically to:

```text
helax-homelab/homelab-toolbox
```

For generic GitHub repository authentication, use [helax-homelab-actions/git-url-auth](https://github.com/helax-homelab-actions/git-url-auth) directly.
