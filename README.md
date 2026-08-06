# Shared Github Actions workflows

This repository contains reusable Github Actions workflows that can be called from other repositories.

## Workflows

### Build and deploy workflows

- `docker-build-push.yaml` - Builds and pushes a Docker image to GHCR
- `java-gradle-build-push.yaml` - Builds and pushes a Java application to GHCR
- `java-kustomize-deploy.yaml` - Deploys a Java application to AKS using Kustomize

### Release workflows

- `create-release-draft.yaml` - Creates or updates a `vM.N` draft release (starting at `v0.0`, incrementing the minor version, and updating an existing draft instead of duplicating it) with all commits since the previous `vM.N` release, triggered on push to `main`
- `publish-major-version.yaml` - Publishes or updates a floating major-version tag (e.g. `v1`) when a matching minor release (e.g. `v1.7`) is published

## Usage

To use these workflows in another repository, add a workflow file that calls the reusable workflow. For example:

```yaml
on: push

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  packages: write
  contents: read

jobs:

  build-push:
    uses: ./.github/workflows/reusable-build-push.yaml
    with:
      java-version: '8'
    secrets: inherit

  beta:
    needs: build-push
    uses: ./.github/workflows/reusable-deploy.yaml
    strategy:
      matrix:
        org: [ bym-oslo-kommune-no ]
    with:
      tags: ${{ needs.build-push.outputs.tags }}
      environment: beta
      org: ${{ matrix.org }}
      deploy-every-branch: true
    secrets: inherit

  prod:
    needs: [ build-push, beta ]
    uses: ./.github/workflows/reusable-deploy.yaml
    strategy:
      matrix:
        org: [ nfk-no, mrfylke-no ]
    with:
      tags: ${{ needs.build-push.outputs.tags }}
      environment: api
      org: ${{ matrix.org }}
    secrets: inherit
```
