# Shared Github Actions workflows

This repository contains reusable Github Actions workflows that can be called from other repositories.

## Workflows

### Build and push workflows

- `docker-build-push.yaml` - Builds and pushes a Docker image to Github Container Registry
- `java-gradle-build-push.yaml` - Builds and pushes a Java application to Github Container Registry

### Deployment workflows

- `kustomize-deploy.yaml` - Deploys an application to Azure Kubernetes Service using Kustomize

### Release workflows

- `create-release-draft.yaml` - Creates or updates a `vM.N` draft release (starting at `v0.0`, incrementing the minor version, and updating an existing draft instead of duplicating it) with all commits since the previous `vM.N` release, triggered on push to `main`
- `publish-major-version.yaml` - Publishes or updates a floating major-version tag (e.g. `v1`) when a matching minor release (e.g. `v1.7`) is published

### Admin workflows

- `sync-topics-to-teams.yaml` - Nightly sync that finds repos by topic (`arkiv`, `informasjonsmodell`, etc.) and adds them to the corresponding team with `write` permission

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
    uses: ./.github/workflows/java-gradle-build-push.yaml
    with:
      java-version: '8'
    secrets: inherit

  beta:
    needs: build-push
    uses: ./.github/workflows/kustomize-deploy.yaml
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
    uses: ./.github/workflows/kustomize-deploy.yaml
    strategy:
      matrix:
        org: [ nfk-no, mrfylke-no ]
    with:
      tags: ${{ needs.build-push.outputs.tags }}
      environment: api
      org: ${{ matrix.org }}
    secrets: inherit
```
