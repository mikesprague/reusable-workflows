# reusable-workflows

Experimenting with reusable workflows ([official docs](https://docs.github.com/en/actions/using-workflows/reusing-workflows))

WIP - use at your own risk :sweat_smile:

## Workflows

### Dependabot Auto-merge

Runs against Dependabot PRs and performs a rebase merge if:

- PR is for a minor release of a dev dependency
- PR is for a patch release of any dependency

Example usage:

```yaml
name: 🤖 Dependabot auto-merge

on: [pull_request]

permissions:
  contents: write
  pull-requests: write

jobs:
  call-auto-merge-workflow:
    if: ${{ github.actor == 'dependabot[bot]' }}
    uses: mikesprague/reusable-workflows/.github/workflows/dependabot-auto-merge.yml@main
    secrets:
      TOKEN: ${{ secrets.GITHUB_TOKEN}}
```

### Deploy to GitHub Pages

Deploys a folder (or file) to GitHub Pages, requires a build artifact reference to deploy

Example step to build artifact (this step should be at end of a build job):

```yaml
    - name: 📦 Create and upload build artifact
      uses: actions/upload-artifact@v3
      with:
        name: build-artifact
        path: 'dist/'
        retention-days: 1
```

Example usage (build job steps not included in example):

```yaml
name: Deploy

on:
  push:
    branches:
      - "main"

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    - your existing
    - build steps
    # create build artifact for reusable workflow
    - name: 📦 Create and upload build artifact
      uses: actions/upload-artifact@v3
      with:
        name: build-artifact
        path: 'dist/'
        retention-days: 1

  call-deploy-workflow:
    needs: [build]
    uses: mikesprague/reusable-workflows/.github/workflows/pages-deploy.yml@main
    secrets:
      REPO_TOKEN: ${{ secrets.GITHUB_TOKEN}}
    with:
      artifact-name: build-artifact
```

### Create Conventional Commit Release

Creates a GitHub release for a strict semver tag (`vX.Y.Z`) using Conventional Commits.

Supports optional prerelease/draft releases and a custom changelog config path.

Example usage:

```yaml
name: 🚀 Release

on:
  push:
    tags:
      - 'v*.*.*'

permissions:
  contents: write

jobs:
  call-release-workflow:
    uses: mikesprague/reusable-workflows/.github/workflows/create-release.yml@main
    secrets:
      REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    with:
      prerelease: false
      draft: false
      release-notes-heading: "What's Changed"
      changelog-preset: conventionalcommits
      changelog-config-path: changelog.config.mjs
```

### Deploy to Fly.io

Deploys your app to Fly.io using `flyctl deploy`.

Supports optional region/strategy/build target/build path overrides.

Example usage:

```yaml
name: 🚀 Deploy to Fly.io

on:
  push:
    branches:
      - "main"

permissions:
  contents: read

jobs:
  call-flyio-deploy-workflow:
    uses: mikesprague/reusable-workflows/.github/workflows/flyio-deploy.yml@main
    secrets:
      REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
    with:
      region: ewr
      strategy: immediate
      build-target: production
      build-path: ./
```

### Run Lighthouse Report

Runs a Lighthouse audit against a URL and publishes the report URL on the job environment.

Requires a target URL and supports an optional environment name.

Example usage:

```yaml
name: 🗼 Lighthouse Report

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  call-lighthouse-workflow:
    uses: mikesprague/reusable-workflows/.github/workflows/lighthouse.yml@main
    secrets:
      REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    with:
      url: https://example.com
      environment-name: lighthouse-report
```
