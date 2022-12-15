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

### Deploy to Fly.io

... documentation in progress ...

### Run Lighthouse Report

... documentation in progress ...
