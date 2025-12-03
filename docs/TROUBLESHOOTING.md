# Troubleshooting

This guide covers common issues and their solutions when using `action-helm-deploy`.

## Table of Contents

- [Authentication Errors](#authentication-errors)
- [Chart Not Found Errors](#chart-not-found-errors)
- [Kubectl Errors](#kubectl-errors)
- [Helm Template Errors](#helm-template-errors)

## Authentication Errors

### Error: `gcloud auth print-access-token` fails

**Symptoms:**
```
Error: PERMISSION_DENIED: Permission denied on resource
```

**Solutions:**

1. Ensure you have authenticated to GCP before using this action:

```yaml
- name: Authenticate to Google Cloud
  uses: google-github-actions/auth@v2
  with:
    credentials_json: ${{ secrets.GCP_SA_KEY }}

- name: Set up Cloud SDK
  uses: google-github-actions/setup-gcloud@v2
```

2. Verify your service account has the required roles:
   - `roles/artifactregistry.reader` - to pull charts from Artifact Registry

### Error: `helm registry login` fails

**Symptoms:**
```
Error: failed to authorize: failed to fetch oauth token
```

**Solutions:**

1. Verify the region format is correct (e.g., `europe-west2`, not `europe-west2-a`)
2. Ensure the Artifact Registry repository exists in the specified region
3. Check that the service account has access to the repository

## Chart Not Found Errors

### Error: `chart not found`

**Symptoms:**
```
Error: chart "oci://region-docker.pkg.dev/project/repo/chart" not found
```

**Solutions:**

1. Verify the chart exists in the registry:
   ```bash
   gcloud artifacts docker images list \
     REGION-docker.pkg.dev/PROJECT_ID/REPOSITORY_NAME
   ```

2. Check that all parameters are correct:
   - `region`: Must match where the chart is stored
   - `gcp_project_id`: Must be the correct project ID
   - `repository_name`: Must match the Artifact Registry repository name
   - `chart_name`: Must match the chart name exactly
   - `chart_version`: Must be a valid, existing version

3. Ensure the chart was pushed as an OCI artifact:
   ```bash
   helm push chart-1.0.0.tgz oci://REGION-docker.pkg.dev/PROJECT_ID/REPOSITORY_NAME
   ```

## Kubectl Errors

### Error: `connection refused` or `unable to connect to the server`

**Symptoms:**
```
Unable to connect to the server: dial tcp: lookup kubernetes.default.svc.cluster.local
```

**Solutions:**

1. Configure kubectl before using this action:

```yaml
- name: Configure kubectl
  run: |
    gcloud container clusters get-credentials CLUSTER_NAME \
      --region REGION \
      --project ${{ secrets.GCP_PROJECT_ID }}
```

2. Ensure the GKE auth plugin is installed:

```yaml
- name: Install gke-gcloud-auth-plugin
  run: |
    gcloud components install gke-gcloud-auth-plugin
```

### Error: `forbidden` or RBAC errors

**Symptoms:**
```
Error from server (Forbidden): deployments.apps is forbidden
```

**Solutions:**

1. Ensure the service account has Kubernetes RBAC permissions
2. Check that the service account has the `roles/container.developer` role or equivalent
3. Verify cluster-level RBAC allows the operations in your Helm chart

## Helm Template Errors

### Error: `values file not found`

**Symptoms:**
```
Error: open ./values.yaml: no such file or directory
```

**Solutions:**

1. Ensure the values file path is relative to your repository root
2. Verify the file exists and is committed to your repository
3. Check for typos in the `chart_value_file` parameter

### Error: `template rendering failed`

**Symptoms:**
```
Error: template: chart/templates/deployment.yaml:10:12: executing "chart/templates/deployment.yaml" at <.Values.somefield>: nil pointer evaluating interface {}.somefield
```

**Solutions:**

1. Verify all required values are defined in your values file
2. Check that the action-injected values (`appVersion`, `environment`, `gcpProjectId`, `region`) are handled correctly in your templates
3. Use default values in templates for optional fields:
   ```yaml
   {{ .Values.appVersion | default "latest" }}
   ```

## Getting More Help

If your issue is not covered here:

1. Check the [GitHub Issues](https://github.com/martoc/action-helm-deploy/issues) for similar problems
2. Enable debug logging in your workflow:
   ```yaml
   env:
     ACTIONS_STEP_DEBUG: true
   ```
3. Open a new issue with:
   - Your workflow configuration (redact secrets)
   - The full error message
   - Steps to reproduce the issue
