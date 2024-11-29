# Usage

* Install a Helm chart into a GKE cluster.

```yaml
- name: Deploy
  uses: martoc/action-helm-deploy@v0
  with:
    registry: gcp
    region: europe-west2
    repository_name: repository-name
    gcp_project_id: project-id
    workload_identity_provider: workload_identity_provider
    service_account: service_account
    cluster_name: your-cluster_name
    chart_name: your-chart
    chart_version: your-chart-version
    chart_values_file: your-values
```
