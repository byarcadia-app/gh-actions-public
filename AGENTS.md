# gh-actions-public

Public reusable GitHub Actions workflows for byArcadia (`byarcadia-app` org).

## Commands

```bash
actionlint    # Lint all workflows — must pass before merge
```

## Overview

Public counterpart to `gh-actions` (private). Workflows that need to be callable from public repositories live here, since GitHub does not allow public repos to use reusable workflows from private repos. Callers reference via `uses: byarcadia-app/gh-actions-public/.github/workflows/<name>@main`.

## Structure

```
gh-actions-public/
├── .github/
│   ├── actionlint.yaml              # actionlint config (no self-hosted runners)
│   └── workflows/
│       ├── ci-actionlint.yaml       # CI: lints workflows on PR and push to main
│       └── sync-skill-to-hub.yaml   # Reusable: push skill from source repo → hub PR
├── .gitignore
├── CLAUDE.md
├── CONTRIBUTING.md
└── README.md
```

## Workflows

### Skill sync

- **sync-skill-to-hub** (push-based) — Source repo pushes a skill to the hub. Checks out both source and hub repos, copies skill directory, optionally formats (default: `deno fmt`), creates/updates PR in hub. Requires `target-repo-token` secret with write access to hub. Inputs: `skill-name`, `skill-path`, `source-ref`, `target-repo`, `target-branch`, `target-skills-path`, `format-command`, `pr-labels`. Outputs: `changed`, `pr-number`, `pr-url`.

### CI

- **ci-actionlint** — Internal CI for this repo. Runs `actionlint` via Docker on PRs and pushes to main. Ignores SC2086 and SC2129 shellcheck warnings.

## Conventions

- All workflows use `ubuntu-latest` runners (no self-hosted).
- Reusable workflows do NOT set concurrency — the caller controls that.
- Skill names must be kebab-case (`^[a-z0-9]([a-z0-9-]*[a-z0-9])?$`).
- Input validation includes path traversal prevention for skill paths.
- PR branches follow `sync/skill-{name}` naming.
- Existing PRs are reused via force-push to the sync branch.
- Committer identity: `github-actions[bot]`.
- Workflow files use `.yaml` extension (not `.yml`).
- Hub CI requires `deno fmt` — synced files are auto-formatted before commit.
