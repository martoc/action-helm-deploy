# Usage

## Overview

This GitHub Action is designed to deploy Helm charts to Kubernetes clusters.
It simplifies the CI/CD workflow by automating the process of pulling Helm charts
from a registry and deploying them to a Kubernetes cluster.

## Prerequisites

Secrets Configuration

Before using this action, ensure that you have the following secrets configured in your GitHub repository:

### GCP Artifact Registry

* **GCP_WORKLOAD_IDENTITY_PROVIDER:** Workload identity provider for GCP authentication.
* **GCP_SERVICE_ACCOUNT:** The email of the service account to use for GCP authentication.

## Inputs

### Action Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `registry` | Registry to pull the Helm chart from. Valid values: `gcp` | No | `docker.io` |
| `region` | Region where the registry is located. Valid for GCP regions | No | `""` |
| `repository_name` | Repository name in the registry | No | `""` |
| `environment` | Environment name for deployment | No | `""` |
| `gcp_project_id` | Google Cloud Project ID | No | `""` |
| `chart_name` | Name of the Helm chart to deploy | Yes | - |
| `chart_version` | Version of the Helm chart to deploy | Yes | - |
| `chart_value_file` | Path to the Helm values file | Yes | - |
| `version` | Application version to set in the deployment | No | `""` |

### Environment Variables

The following environment variables can be used to customize the deployment:

| Variable | Description | Required |
|----------|-------------|----------|
| `TAG_VERSION` | The version tag for the application (used if `version` input is not provided) | No |

## Example

Here are examples of how to use this GitHub Action to deploy Helm charts
to GKE clusters.

```yaml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  gcp:
    permissions:
      contents: write
      id-token: write
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 50
          fetch-tags: true
      - name: Tag
        uses: martoc/action-tag@v0
        with:
          skip-push: true
      - uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
          service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
      - uses: google-github-actions/get-gke-credentials@v2
        with:
          cluster_name: your-cluster
          location: europe-west2
      - name: Deploy
        uses: martoc/action-helm-deploy@v0
        with:
          registry: gcp
          region: europe-west2
          repository_name: repository
          gcp_project_id: project-id
          chart_name: your-chart
          chart_version: 1.0.0
          chart_value_file: values.yaml
          environment: production

  with-version:
    permissions:
      contents: write
      id-token: write
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 50
          fetch-tags: true
      - uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
          service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
      - uses: google-github-actions/get-gke-credentials@v2
        with:
          cluster_name: your-cluster
          location: europe-west2
      - name: Deploy with specific version
        uses: martoc/action-helm-deploy@v0
        with:
          registry: gcp
          region: europe-west2
          repository_name: repository
          gcp_project_id: project-id
          chart_name: your-chart
          chart_version: 1.0.0
          chart_value_file: values.yaml
          version: 2.0.0
```

## Registry-Specific Requirements

### GCP Artifact Registry

* `registry` input must be set to `gcp`
* `region` input is required
* `repository_name` input is required
* `gcp_project_id` input is required
* GCP authentication must be configured (e.g., using `google-github-actions/auth`)
* GKE credentials must be configured (e.g., using `google-github-actions/get-gke-credentials`)

## Helm Values

The action automatically sets the following values when deploying the chart:

* `appVersion` - Set to `TAG_VERSION` environment variable or `version` input
* `environment` - Set to the `environment` input value
* `gcpProjectId` - Set to the `gcp_project_id` input value
* `region` - Set to the `region` input value
