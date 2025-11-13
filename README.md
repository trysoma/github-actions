# Pulumi GitHub Actions

A collection of reusable GitHub Actions for Pulumi workflows, providing standardized CI/CD patterns for Pulumi repositories.

## Available Actions

### 1. [pulumi-check](./pulumi-check)
Validates and previews Pulumi changes for pull requests.
- Dependency installation
- Stack selection
- Refresh
- Preview generation
- PR comment posting

### 2. [pulumi-deploy](./pulumi-deploy)
Deploys Pulumi infrastructure with safety checks and multiple operation modes.
- Preview/Up/Destroy operations
- Production safety guards
- Auto-approve options
- GitHub summaries

### 3. [changie-create-pr](./changie-create-pr)
Creates PRs with version bumps and changelog updates.
- Automated changelog batching
- Version file updates
- Creates release PR

### 4. [changie-release](./changie-release)
Creates git tags and GitHub releases (works with changie-create-pr).
- Reads version from files
- Git tag creation
- Comprehensive release notes
- Skips if tag exists

## Installation

### Method 1: Direct Reference (Recommended)

Reference actions directly from this repository in your workflows:

```yaml
- uses: faro-engineering/github-actions/pulumi-check@v1
  with:
    stack: dev
    pulumi-access-token: ${{ secrets.PULUMI_ACCESS_TOKEN }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Method 2: Copy to Your Repository

Copy the action directories to your repository's `.github/actions/` folder.

## Quick Start

### For PR Checks

```yaml
name: Pulumi PR Check

on:
  pull_request:
    paths:
      - '**.ts'
      - '**.py'
      - '**.go'
      - '**/Pulumi.*.yaml'

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: faro-engineering/github-actions/pulumi-check@v1
        with:
          stack: dev
          environment-name: Development
          pulumi-access-token: ${{ secrets.PULUMI_ACCESS_TOKEN }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### For Deployments

```yaml
name: Deploy Infrastructure

on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [dev, staging, prod]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: faro-engineering/github-actions/pulumi-deploy@v1
        with:
          project: my-app
          environment: ${{ inputs.environment }}
          pulumi-access-token: ${{ secrets.PULUMI_ACCESS_TOKEN }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          auto-approve: true
```

### For Automated Releases

**Step 1: Create changelog PR when PRs are merged**

```yaml
name: Create Changelog PR

on:
  pull_request:
    types: [closed]
    branches: [main]

jobs:
  changelog:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.BOT_PAT }}
      - uses: faro-engineering/github-actions/changie-create-pr@v1
```

**Step 2: Create release when VERSION changes**

```yaml
name: Create Release

on:
  push:
    branches: [main]
    paths:
      - 'VERSION'

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.BOT_PAT }}
      - uses: faro-engineering/github-actions/changie-release@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          bot-token: ${{ secrets.BOT_PAT }}
```

## Required Secrets

| Secret | Description | Used By |
|--------|-------------|---------|
| `PULUMI_ACCESS_TOKEN` | Pulumi access token with preview/up permissions | pulumi-check, pulumi-deploy |
| `GITHUB_TOKEN` | Automatically provided by GitHub Actions | All actions |
| `BOT_PAT` | Personal Access Token for bot user with branch bypass permissions | changie-create-pr, changie-release |

## Documentation

- [Pulumi Check Action](./docs/pulumi-check.md)
- [Pulumi Deploy Action](./docs/pulumi-deploy.md)
- [Changie Release Action](./docs/changie-release.md)
- [Examples](./examples)

## Support

For issues or questions, please open an issue in this repository.

## License

MIT
