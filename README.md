[![checks](https://github.com/martoc/action-helm-deploy/actions/workflows/checks.yml/badge.svg?branch=main&event=push)](https://github.com/martoc/action-helm-deploy/actions/workflows/checks.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![slack](https://img.shields.io/badge/slack-general-brightgreen.svg?logo=slack)](https://app.slack.com/messages/T8L8AAD3M/C8LBHLSVA)

# action-helm-deploy

A GitHub Action that deploys Helm charts from OCI registries to Kubernetes clusters. This action renders Helm templates and applies the resulting Kubernetes manifests using `kubectl`.

## Features

- Deploy Helm charts from Google Cloud Artifact Registry (OCI)
- Automatic authentication with GCP using `gcloud`
- Pass custom values and environment-specific configurations
- Support for versioned deployments with `appVersion`
- Generates and applies Kubernetes manifests in a single step

## Prerequisites

Before using this action, ensure the following tools are available in your workflow:

| Tool | Purpose |
|------|---------|
| `gcloud` | GCP authentication and access token generation |
| `helm` | Helm chart templating |
| `kubectl` | Kubernetes manifest deployment |

You must also have:

- A GCP service account with permissions to read from Artifact Registry
- `kubectl` configured with access to your target Kubernetes cluster
- Prior authentication step using `google-github-actions/auth`

## Quick Start

```yaml
- name: Authenticate to Google Cloud
  uses: google-github-actions/auth@v2
  with:
    credentials_json: ${{ secrets.GCP_SA_KEY }}

- name: Set up Cloud SDK
  uses: google-github-actions/setup-gcloud@v2

- name: Deploy Helm Chart
  uses: martoc/action-helm-deploy@v0
  with:
    registry: gcp
    region: europe-west2
    repository_name: helm-charts
    gcp_project_id: my-project-id
    chart_name: my-application
    chart_version: 1.0.0
    chart_value_file: ./values/production.yaml
    environment: production
    version: ${{ github.sha }}
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `registry` | No | `docker.io` | Registry type. Currently only `gcp` is supported for deployments |
| `region` | No | `""` | GCP region (e.g., `europe-west2`, `us-central1`) |
| `repository_name` | No | `""` | Name of the Artifact Registry repository |
| `gcp_project_id` | No | `""` | Google Cloud Project ID |
| `chart_name` | **Yes** | - | Name of the Helm chart to deploy |
| `chart_version` | **Yes** | - | Version of the Helm chart |
| `chart_value_file` | **Yes** | - | Path to the Helm values file |
| `environment` | No | `""` | Environment name (e.g., `dev`, `staging`, `production`) |
| `version` | No | `""` | Application version, passed as `appVersion` to the Helm chart |

## How It Works

1. **Authentication**: Logs into the GCP Artifact Registry using `gcloud auth print-access-token`
2. **Template Rendering**: Uses `helm template` to render the chart with:
   - Values from the specified values file
   - Custom values: `appVersion`, `environment`, `gcpProjectId`, `region`
3. **Deployment**: Applies the generated `deployment.yaml` manifest using `kubectl apply`

## Helm Values

The action automatically sets the following values when rendering the Helm template:

```yaml
appVersion: <version input>
environment: <environment input>
gcpProjectId: <gcp_project_id input>
region: <region input>
```

Ensure your Helm chart templates can consume these values.

## Documentation

For more detailed documentation, see:

- [Usage Examples](./docs/USAGE.md)
- [Code Style](./docs/CODESTYLE.md)
- [Contributing](./.github/CONTRIBUTING.md)
- [Security Policy](./.github/SECURITY.md)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
