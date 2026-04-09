# gh-actions-public

Public reusable GitHub Actions workflows for [byArcadia](https://github.com/byarcadia-app) projects.

This is the public counterpart to [`gh-actions`](https://github.com/byarcadia-app/gh-actions) (private). Workflows that need to be called from public repositories live here, since GitHub does not allow public repos to use reusable workflows from private repos.

## Reusable Workflows

| Workflow | Description | Caller trigger |
|---|---|---|
| [sync-skill-to-hub](.github/workflows/sync-skill-to-hub.yaml) | Push a skill from source repo to hub (creates PR) | `push` |

## Usage

### Sync skill to hub (push-based)

In a source repo (e.g. `byarcadia-app/plutus`), create `.github/workflows/sync-skills-to-hub.yaml`:

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

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
