# GitHub Workflows

Reusable GitHub Actions workflows for @ivoronin projects.

## Workflows

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `test.yml` | `workflow_call` | Run `make test-all` |
| `release.yml` | `workflow_call` | Build release, publish to brew/docker/krew |
| `autorelease.yml` | `workflow_call` | Data-embedding tools: `make update` → PR → auto-merge → CalVer tag → release |

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

### autorelease.yml (data-embedding projects)

Regenerates embedded data on a schedule; if it changed, opens a PR that auto-merges once
`test / test` passes, then tags a CalVer release (which triggers `release.yml`). Requires
repo auto-merge enabled and a `PAT_TOKEN` secret.

```yaml
name: Auto-release
on:
  schedule:
    - cron: '0 0 * * 0'  # weekly
  workflow_dispatch:
  pull_request_target:
    types: [closed]
jobs:
  autorelease:
    uses: ivoronin/github-workflows/.github/workflows/autorelease.yml@main
    with:
      language: go  # or python or swift
    secrets: inherit  # requires PAT_TOKEN
```

## Notes

- Swift workflows run on `macos-latest` where Swift and SwiftLint are pre-installed.

## Required Secrets

| Secret | Required For | Purpose |
|--------|--------------|---------|
| `HOMEBREW_TOKEN` | `release.yml` (brew: true) | Push to Homebrew tap |
| `PAT_TOKEN` | `autorelease.yml` | Create/merge the data PR and push the release tag |

## Requirements

Projects using these workflows must have:
- `Makefile` with `test-all` and `release` targets
- `make update` target and repo auto-merge enabled (for `autorelease.yml`)
- Branch protection requiring the `test / test` status check
