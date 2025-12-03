# Usage

This document provides usage examples for the `action-helm-deploy` GitHub Action.

## Table of Contents

- [Basic GCP Deployment](#basic-gcp-deployment)
- [Full Workflow Example](#full-workflow-example)
- [Multi-Environment Deployment](#multi-environment-deployment)
- [Using with Version Tags](#using-with-version-tags)
- [Values File Examples](#values-file-examples)

## Basic GCP Deployment

Deploy a Helm chart from GCP Artifact Registry:

```yaml
- name: Deploy
  uses: martoc/action-helm-deploy@v0
  with:
    registry: gcp
    region: europe-west2
    repository_name: helm-charts
    gcp_project_id: my-project-id
    chart_name: my-application
    chart_version: 1.0.0
    chart_value_file: ./values.yaml
```

## Full Workflow Example

A complete workflow that authenticates to GCP and deploys a Helm chart:

```yaml
name: Deploy to GKE

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}

      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v2

      - name: Install gke-gcloud-auth-plugin
        run: |
          gcloud components install gke-gcloud-auth-plugin

      - name: Configure kubectl
        run: |
          gcloud container clusters get-credentials CLUSTER_NAME \
            --region REGION \
            --project ${{ secrets.GCP_PROJECT_ID }}

      - name: Deploy Helm Chart
        uses: martoc/action-helm-deploy@v0
        with:
          registry: gcp
          region: europe-west2
          repository_name: helm-charts
          gcp_project_id: ${{ secrets.GCP_PROJECT_ID }}
          chart_name: my-application
          chart_version: 1.0.0
          chart_value_file: ./kubernetes/values.yaml
          environment: production
          version: ${{ github.sha }}
```

## Multi-Environment Deployment

Deploy to different environments using separate values files:

### Development

```yaml
- name: Deploy to Development
  uses: martoc/action-helm-deploy@v0
  with:
    registry: gcp
    region: europe-west2
    repository_name: helm-charts
    gcp_project_id: my-project-dev
    chart_name: my-application
    chart_version: 1.0.0
    chart_value_file: ./values/dev.yaml
    environment: dev
    version: ${{ github.sha }}
```

### Staging

```yaml
- name: Deploy to Staging
  uses: martoc/action-helm-deploy@v0
  with:
    registry: gcp
    region: europe-west2
    repository_name: helm-charts
    gcp_project_id: my-project-staging
    chart_name: my-application
    chart_version: 1.0.0
    chart_value_file: ./values/staging.yaml
    environment: staging
    version: ${{ github.sha }}
```

### Production

```yaml
- name: Deploy to Production
  uses: martoc/action-helm-deploy@v0
  with:
    registry: gcp
    region: europe-west2
    repository_name: helm-charts
    gcp_project_id: my-project-prod
    chart_name: my-application
    chart_version: 1.0.0
    chart_value_file: ./values/production.yaml
    environment: production
    version: ${{ github.ref_name }}
```

## Using with Version Tags

Deploy specific versions based on Git tags:

```yaml
- name: Deploy Release
  uses: martoc/action-helm-deploy@v0
  with:
    registry: gcp
    region: us-central1
    repository_name: helm-charts
    gcp_project_id: my-project-id
    chart_name: my-application
    chart_version: ${{ github.ref_name }}
    chart_value_file: ./values/production.yaml
    environment: production
    version: ${{ github.ref_name }}
```

## Values File Examples

### Minimal Values File

```yaml
# values.yaml
replicaCount: 1

image:
  repository: my-app
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
```

### Production Values File

```yaml
# values/production.yaml
replicaCount: 3

image:
  repository: my-app
  pullPolicy: Always

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: app.example.com
      paths:
        - path: /
          pathType: Prefix
```

### Using Action-Injected Values

The action automatically injects certain values that you can use in your templates:

```yaml
# In your Helm template (e.g., deployment.yaml)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  labels:
    app.kubernetes.io/version: {{ .Values.appVersion | quote }}
    environment: {{ .Values.environment | quote }}
spec:
  template:
    metadata:
      annotations:
        gcpProjectId: {{ .Values.gcpProjectId | quote }}
        region: {{ .Values.region | quote }}
```
