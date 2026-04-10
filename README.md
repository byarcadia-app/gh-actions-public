# gh-actions-public

Public reusable GitHub Actions workflows for [byArcadia](https://github.com/byarcadia-app) projects.

This is the public counterpart to [`gh-actions`](https://github.com/byarcadia-app/gh-actions) (private). Workflows that need to be called from public repositories live here, since GitHub does not allow public repos to use reusable workflows from private repos.

## Reusable Workflows

| Workflow | Description |
|---|---|
| [sync-skill-to-hub](#sync-skill-to-hub) | Push a skill from source repo to hub via PR |

---

## sync-skill-to-hub

PUSH-based workflow that syncs a skill directory from a source repository to [hub](https://github.com/byarcadia-app/hub). Creates or updates a pull request in the target repository when changes are detected.

Source repos (e.g. `plutus`, `aether-ui`) maintain their own Claude Code skills in `skills/`. The `hub` repo aggregates all skills into one registry. This workflow automates the sync — on every push to `skills/**` it copies the skill to hub and opens a PR.

### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `skill-name` | Skill name (kebab-case, e.g. `aether-setup`) | yes | — |
| `skill-path` | Path to skill directory in the source repo | yes | — |
| `source-ref` | Source repo ref to checkout (empty = default branch) | no | `""` |
| `target-repo` | Target hub repository (`owner/repo`) | no | `byarcadia-app/hub` |
| `target-branch` | Target hub base branch | no | `main` |
| `target-skills-path` | Directory in hub where skills live | no | `skills` |
| `format-command` | Format command to run on synced skill (empty to skip) | no | `deno fmt` |
| `pr-labels` | Comma-separated labels to add to the PR | no | `""` |

### Secrets

| Name | Description | Required |
|------|-------------|----------|
| `target-repo-token` | Token with write access to the target hub repo | yes |

### Outputs

| Name | Description |
|------|-------------|
| `changed` | Whether changes were detected (`true`/`false`) |
| `pr-number` | PR number (empty if no PR created) |
| `pr-url` | PR URL (empty if no PR created) |

### How it works

1. Validates inputs (kebab-case name, path traversal protection)
2. Checks out both source and target (hub) repos
3. Replaces the skill directory (`skills/<name>/`) with source contents
4. Runs format command (default: `deno fmt`) on synced files
5. If changes detected: commits, force-pushes to `sync/skill-<name>` branch
6. Creates a new PR or reuses existing one on that branch
7. Applies labels if configured

### Usage

#### Basic — single skill

```yaml
name: Sync skills to hub

on:
  push:
    branches: [main]
    paths:
      - "skills/**"
  workflow_dispatch:

jobs:
  my-skill:
    uses: byarcadia-app/gh-actions-public/.github/workflows/sync-skill-to-hub.yaml@main
    with:
      skill-name: my-skill
      skill-path: skills/my-skill
    secrets:
      target-repo-token: ${{ secrets.HUB_WRITE_TOKEN }}
```

#### Multiple skills from one repo

Add one job per skill:

```yaml
jobs:
  aether-setup:
    uses: byarcadia-app/gh-actions-public/.github/workflows/sync-skill-to-hub.yaml@main
    with:
      skill-name: aether-setup
      skill-path: skills/aether-setup
      pr-labels: "automated,sync"
    secrets:
      target-repo-token: ${{ secrets.HUB_WRITE_TOKEN }}

  aether-ui:
    uses: byarcadia-app/gh-actions-public/.github/workflows/sync-skill-to-hub.yaml@main
    with:
      skill-name: aether-ui
      skill-path: skills/aether-ui
      pr-labels: "automated,sync"
    secrets:
      target-repo-token: ${{ secrets.HUB_WRITE_TOKEN }}
```

#### Custom format command

```yaml
jobs:
  my-skill:
    uses: byarcadia-app/gh-actions-public/.github/workflows/sync-skill-to-hub.yaml@main
    with:
      skill-name: my-skill
      skill-path: skills/my-skill
      format-command: ""  # skip formatting
    secrets:
      target-repo-token: ${{ secrets.HUB_WRITE_TOKEN }}
```

See [`plutus`](https://github.com/byarcadia-app/plutus/blob/main/.github/workflows/sync-skills-to-hub.yaml) and [`aether-ui`](https://github.com/byarcadia-app/aether-ui/blob/main/.github/workflows/sync-skills-to-hub.yaml) for real examples.

---

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

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
