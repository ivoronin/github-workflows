# GitHub Workflows

Reusable GitHub Actions workflows for @ivoronin projects.

## Workflows

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `test.yml` | `workflow_call` | Run `make test-all` |
| `release.yml` | `workflow_call` | Build release, publish to brew/docker/krew |
| `autoupdate.yml` | `workflow_call` | Run `make update`, create PR |
| `automerge.yml` | `workflow_call` | Auto-merge Dependabot/autoupdate PRs |
| `autotag.yml` | `workflow_call` | Create CalVer tag after PR merge |

## Usage

### test.yml

```yaml
name: Test
on:
  push:
    branches: [main]
  pull_request:
jobs:
  test:
    uses: ivoronin/github-workflows/.github/workflows/test.yml@main
    with:
      language: go  # or python or swift
```

### release.yml

```yaml
name: Release
on:
  push:
    tags: ['v*']
jobs:
  release:
    uses: ivoronin/github-workflows/.github/workflows/release.yml@main
    with:
      language: go    # or python or swift
      docker: false   # Build and push to GHCR
      brew: true      # Update Homebrew tap
      krew: false     # Update Krew index
    secrets: inherit
```

### automerge.yml

```yaml
name: Auto-merge
on:
  pull_request:
    types: [opened, synchronize, reopened, labeled]
jobs:
  automerge:
    uses: ivoronin/github-workflows/.github/workflows/automerge.yml@main
    with:
      dependabot: true   # Auto-merge Dependabot minor/patch
      autoupdate: false  # Auto-merge auto-update PRs
    secrets: inherit
```

### autoupdate.yml (autorelease projects)

```yaml
name: Auto-update
on:
  schedule:
    - cron: '0 0 * * 0'  # weekly
  workflow_dispatch:
jobs:
  autoupdate:
    uses: ivoronin/github-workflows/.github/workflows/autoupdate.yml@main
    with:
      language: go  # or python or swift
    secrets: inherit
```

### autotag.yml (autorelease projects)

```yaml
name: Auto-tag
on:
  pull_request:
    types: [closed]
jobs:
  autotag:
    uses: ivoronin/github-workflows/.github/workflows/autotag.yml@main
    with:
      dependabot: false
      autoupdate: true
    secrets: inherit  # Requires PAT_TOKEN secret
```

## Notes

- Swift workflows run on `macos-latest` where Swift and SwiftLint are pre-installed.

## Required Secrets

| Secret | Required For | Purpose |
|--------|--------------|---------|
| `HOMEBREW_TOKEN` | `release.yml` (brew: true) | Push to Homebrew tap |
| `PAT_TOKEN` | `autotag.yml` | Push tags that trigger release workflow |

## Requirements

Projects using these workflows must have:
- `Makefile` with `test-all` and `release` targets
- `make update` target (for autoupdate workflow)
- Branch protection requiring `test` status check (for automerge)
