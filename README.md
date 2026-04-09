# gh-actions-public

Public reusable GitHub Actions workflows for [byArcadia](https://github.com/byarcadia-app) projects.

This is the public counterpart to [`gh-actions`](https://github.com/byarcadia-app/gh-actions) (private). Workflows that need to be called from public repositories live here, since GitHub does not allow public repos to use reusable workflows from private repos.

## Reusable Workflows

| Workflow | Description | Caller trigger |
|---|---|---|
| [sync-skill-to-hub](.github/workflows/sync-skill-to-hub.yaml) | Push a skill from source repo to hub (creates PR) | `push` |

## What this solves

Source repos (e.g. `plutus`, `aether`) maintain their own Claude Code skills in `skills/`. The [`hub`](https://github.com/byarcadia-app/hub) repo aggregates all skills into one registry. The `sync-skill-to-hub` workflow automates this — on every push to `skills/**` it copies the skill to hub and opens a PR.

## Setup

### 1. Create a Personal Access Token

The workflow needs a token with write access to the [`hub`](https://github.com/byarcadia-app/hub) repo to push branches and create PRs.

Create a **Fine-grained PAT** at [github.com/settings/personal-access-tokens/new](https://github.com/settings/personal-access-tokens/new):

- **Resource owner**: `byarcadia-app`
- **Repository access**: select `hub` only
- **Permissions**: `Contents: Read and write`, `Pull requests: Read and write`

> If `byarcadia-app` is not available as resource owner, an org admin needs to enable fine-grained PATs at `https://github.com/organizations/byarcadia-app/settings/personal-access-tokens`. Alternatively, use a Classic token with `repo` scope.

### 2. Add the token as a repository secret

```bash
gh secret set HUB_WRITE_TOKEN --repo byarcadia-app/<your-repo>
```

### 3. Create the caller workflow

In your source repo, create `.github/workflows/sync-skills-to-hub.yaml`:

```yaml
name: Sync skills to hub

on:
  push:
    branches: [main]
    paths:
      - "skills/**"
  workflow_dispatch:

jobs:
  plutus-setup:
    uses: byarcadia-app/gh-actions-public/.github/workflows/sync-skill-to-hub.yaml@main
    with:
      skill-name: plutus-setup
      skill-path: skills/plutus-setup
    secrets:
      target-repo-token: ${{ secrets.HUB_WRITE_TOKEN }}
```

Add one job per skill. See [`plutus`](https://github.com/byarcadia-app/plutus/blob/main/.github/workflows/sync-skills-to-hub.yaml) and [`aether`](https://github.com/byarcadia-app/aether/blob/main/.github/workflows/sync-skills-to-hub.yaml) for real examples.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
