[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

# action-helm-deploy

A GitHub Action that deploys Helm charts from OCI registries to Kubernetes clusters. This action renders Helm templates and applies the resulting Kubernetes manifests using `kubectl`.

## Features

- Deploy Helm charts from OCI registries (GCP Artifact Registry, AWS ECR)
- Automatic authentication with GCP using `gcloud` or AWS using OIDC
- Pass custom values and environment-specific configurations
- Support for versioned deployments with `appVersion`
- Generates and applies Kubernetes manifests in a single step

## Prerequisites

Before using this action, ensure the following tools are available in your workflow:

| Tool | Purpose |
|------|---------|
| `gcloud` | GCP authentication and access token generation (for GCP registry) |
| `aws` | AWS CLI for ECR authentication (for AWS registry) |
| `helm` | Helm chart templating |
| `kubectl` | Kubernetes manifest deployment |

### For GCP Artifact Registry

- A GCP service account with permissions to read from Artifact Registry and access GKE
- Workload Identity Federation configured for GitHub Actions

### For AWS ECR

- An IAM role with permissions to read from ECR, configured for GitHub OIDC
- `kubectl` configured with access to your target Kubernetes cluster (e.g., EKS)

## Quick Start

### GCP Artifact Registry

```yaml
- name: Deploy Helm Chart
  uses: martoc/action-helm-deploy@v0
  with:
    registry: gcp
    region: europe-west2
    repository_name: helm-charts
    gcp_project_id: my-project-id
    workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
    service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
    cluster_name: my-cluster
    chart_name: my-application
    chart_version: 1.0.0
    chart_value_file: ./values/production.yaml
    environment: production
    version: ${{ github.sha }}
```

### AWS ECR

```yaml
- name: Deploy Helm Chart
  uses: martoc/action-helm-deploy@v0
  with:
    registry: aws
    region: eu-west-1
    repository_name: helm-charts
    aws_role_arn: ${{ secrets.AWS_ROLE_ARN }}
    chart_name: my-application
    chart_version: 1.0.0
    chart_value_file: ./values/production.yaml
    environment: production
    version: ${{ github.sha }}
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `registry` | **Yes** | - | Registry type. Valid values: `gcp`, `aws` |
| `region` | **Yes** | - | Cloud region (e.g., `europe-west2`, `eu-west-1`) |
| `repository_name` | **Yes** | - | Name of the registry repository |
| `gcp_project_id` | No | `""` | Google Cloud Project ID (required for GCP registry) |
| `workload_identity_provider` | No | `""` | Workload Identity Provider (required for GCP registry) |
| `service_account` | No | `""` | Service Account email (required for GCP registry) |
| `cluster_name` | No | `""` | Kubernetes cluster name (required for GCP registry) |
| `aws_role_arn` | No | `""` | AWS IAM Role ARN for OIDC authentication (required for AWS registry) |
| `chart_name` | **Yes** | - | Name of the Helm chart to deploy |
| `chart_version` | **Yes** | - | Version of the Helm chart |
| `chart_value_file` | **Yes** | - | Path to the Helm values file |
| `environment` | No | `""` | Environment name (e.g., `dev`, `staging`, `production`) |
| `version` | No | `""` | Application version, passed as `appVersion` to the Helm chart |

## How It Works

### GCP Artifact Registry

1. **Setup**: Sets up Cloud SDK and authenticates using Workload Identity Federation
2. **GKE Credentials**: Configures kubectl with GKE cluster credentials
3. **Registry Login**: Logs into GCP Artifact Registry using `gcloud auth print-access-token`
4. **Template Rendering**: Uses `helm template` to render the chart with:
   - Values from the specified values file
   - Custom values: `appVersion`, `environment`, `gcpProjectId`, `region`
5. **Artifact Storage**: Stores the generated `deployment.yaml` as a workflow artifact (30-day retention)
6. **Deployment**: Applies the generated `deployment.yaml` manifest using `kubectl apply`

### AWS ECR

1. **Authentication**: Configures AWS credentials using OIDC and logs into ECR using `aws ecr get-login-password`
2. **Template Rendering**: Uses `helm template` to render the chart with:
   - Values from the specified values file
   - Custom values: `appVersion`, `environment`, `region`
3. **Artifact Storage**: Stores the generated `deployment.yaml` as a workflow artifact (30-day retention)
4. **Deployment**: Applies the generated `deployment.yaml` manifest using `kubectl apply`

## Helm Values

The action automatically sets the following values when rendering the Helm template:

### GCP Registry

```yaml
appVersion: <version input>
environment: <environment input>
gcpProjectId: <gcp_project_id input>
region: <region input>
```

### AWS Registry

```yaml
appVersion: <version input>
environment: <environment input>
region: <region input>
```

Ensure your Helm chart templates can consume these values.

## Documentation

For more detailed documentation, see:

- [Usage Examples](./docs/USAGE.md)
- [Code Style](./docs/CODESTYLE.md)
- [Troubleshooting](./docs/TROUBLESHOOTING.md)
- [Contributing](./.github/CONTRIBUTING.md)
- [Security Policy](./.github/SECURITY.md)

## Licence

This project is licenced under the MIT Licence - see the [LICENCE](LICENSE) file for details.
