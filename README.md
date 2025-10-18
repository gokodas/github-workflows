# Reusable GitHub Workflows

This repository contains a collection of reusable GitHub Actions workflows designed to streamline CI/CD operations for building, pushing, and deploying containerized applications to Azure services.

## Overview

These workflows are meant to be called as reusable workflows from other repositories using the `uses` keyword in your GitHub Actions workflows. They encapsulate common deployment patterns for Docker container builds and Azure Container Apps deployments.

## Workflows

### 1. Build Docker Image and Push to ACR
**File:** `.github/workflows/build-and-push-to-acr.yml`

Builds a Docker image and pushes it to an Azure Container Registry (ACR).

**Features:**
- Authenticates to Azure using OpenID Connect (OIDC) federation
- Builds Docker images with customizable Dockerfile path and build context
- Supports build arguments (including Sentry auth tokens for sourcemap uploads)
- Tags images with both `latest` and build run number
- Implements Docker layer caching for faster builds
- Pushes built images to specified ACR registry

**Required Inputs:**
- `name`: Name of the job
- `repository`: Docker image repository name
- `registry`: Registry to push the Docker image to
- `subscription_id`: Azure ACR subscription ID

**Optional Inputs:**
- `context`: Path to the Docker context (default: `.`)
- `dockerfile`: Path to the Dockerfile (default: `Dockerfile`)
- `build_args`: Additional build arguments

**Required Secrets:**
- `azure_identity_client_id`: Azure identity client ID
- `azure_entra_id_tenant_id`: Azure Entra ID tenant ID

**Optional Secrets:**
- `sentry_auth_token`: Sentry authentication token for sourcemap uploads

---

### 2. Deploy Application to ACA
**File:** `.github/workflows/deploy-aca.yml`

Deploys a containerized application to Azure Container Apps (ACA).

**Features:**
- Authenticates to Azure using OpenID Connect (OIDC) federation
- Updates an existing ACA with a new Docker image
- Creates a new revision with a versioned suffix
- Polls revision status until the deployment is ready or times out (120 seconds)
- Handles both Running and RunningAtMaxScale states as success
- Provides detailed logging of deployment progress

**Required Inputs:**
- `name`: Name of the app to deploy
- `environment`: Environment to deploy to (e.g., staging, production)
- `image`: Docker image to deploy (full image name with registry)
- `image_tag`: Tag for the Docker image

**Required Secrets:**
- `azure_identity_client_id`: Azure identity client ID
- `azure_entra_id_tenant_id`: Azure Entra ID tenant ID

**Required Variables** (These need to be set in the calling repository for the environment provided via inputs):
- `AZURE_SUBSCRIPTION_ID`: Azure subscription ID
- `AZURE_RESOURCE_GROUP`: Azure resource group name

---

### 3. Deploy Job to ACA
**File:** `.github/workflows/deploy-aca-job.yml`

Deploys and executes a container job in Azure Container Apps (ACA).

**Features:**
- Authenticates to Azure using OpenID Connect (OIDC) federation
- Updates an existing ACA job with a new Docker image
- Triggers job execution immediately after image update
- Polls job execution status until completion (Succeeded/Failed) or times out (120 seconds)
- Provides detailed logging of job progress and execution status

**Required Inputs:**
- `name`: Name of the job to deploy
- `environment`: Environment to deploy to (e.g., staging, production)
- `image`: Docker image to deploy (full image name with registry)
- `image_tag`: Tag for the Docker image

**Required Secrets:**
- `azure_identity_client_id`: Azure identity client ID
- `azure_entra_id_tenant_id`: Azure Entra ID tenant ID

**Required Variables** (These need to be set in the calling repository for the environment provided via inputs):
- `AZURE_SUBSCRIPTION_ID`: Azure subscription ID
- `AZURE_RESOURCE_GROUP`: Azure resource group name

---

## Usage

To use these workflows in your repository, reference them in your GitHub Actions workflow file:

```yaml
jobs:
  build:
    uses: gokodas/github-workflows/.github/workflows/build-and-push-to-acr.yml@v1
    with:
      name: "Build My App"
      repository: "myapp"
      registry: "myregistry.azurecr.io"
      subscription_id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
    secrets:
      azure_identity_client_id: ${{ secrets.AZURE_IDENTITY_CLIENT_ID }}
      azure_entra_id_tenant_id: ${{ secrets.AZURE_ENTRA_ID_TENANT_ID }}
      sentry_auth_token: ${{ secrets.SENTRY_AUTH_TOKEN }} # Optional

  deploy:
    needs: build
    uses: gokodas/github-workflows/.github/workflows/deploy-aca.yml@v1
    with:
      name: "myapp"
      environment: "production"
      image: "myregistry.azurecr.io/myapp"
      image_tag: ${{ github.run_number }}
    secrets:
      azure_identity_client_id: ${{ secrets.AZURE_IDENTITY_CLIENT_ID }}
      azure_entra_id_tenant_id: ${{ secrets.AZURE_ENTRA_ID_TENANT_ID }}
```

## Prerequisites

- Azure subscription with Container Apps enabled
- Azure Container Registry (ACR)
- GitHub repository with configured OIDC federation to Azure
- Required secrets and variables configured in your GitHub repository

## Authentication

All workflows use Azure OpenID Connect (OIDC) federation for secure authentication, eliminating the need to store long-lived credentials in GitHub secrets.
