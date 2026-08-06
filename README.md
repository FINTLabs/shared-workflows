# Shared Github Actions workflows

This repository contains reusable Github Actions workflows that can be called from other repositories.

## Workflows

- `create-release-draft.yaml` - Creates a draft release with all commits since the previous release, triggered on push to `main`
- `docker-build-push.yaml` - Builds and pushes a Docker image to GHCR
- `java-gradle-build-push.yaml` - Builds and pushes a Java application to GHCR
- `java-kustomize-deploy.yaml` - Deploys a Java application to AKS using Kustomize

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
