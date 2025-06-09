# Using Helm Common Chart with FluxCD

This guide explains how to use the Helm Common Chart with FluxCD for GitOps-based deployments.

## Prerequisites

- FluxCD installed and configured in your Kubernetes cluster
- A Git repository for storing your GitOps manifests
- Helm chart repository accessible to FluxCD

## Steps

### 1. Prepare Your Values File

Create a `values.yaml` file with your custom configuration for the Helm chart. This file will be used by FluxCD to deploy the application.

Example `values.yaml`:

```yaml
namespace:
  name: my-app-namespace

containers:
  - name: web
    image:
      repository: nginx
      tag: stable
    env:
      - name: ENVIRONMENT
        value: production
```

### 2. Create a HelmRelease Manifest

Create a `helm-release.yaml` file in your GitOps repository with the following content:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 5m
  chart:
    spec:
      chart: helm-common/helm-common-chart
      sourceRef:
        kind: GitRepository
        name: flux-system
        namespace: flux-system
      version: "0.1.0"
  valuesFrom:
    - kind: Secret
      name: my-app-values
      valuesKey: values.yaml
```

### 3. Create a Kubernetes Secret

Create a Kubernetes secret containing your `values.yaml` file:

```bash
kubectl create secret generic my-app-values --from-file=values.yaml -n flux-system
```

### 4. Commit and Push

Commit the `helm-release.yaml` file to your GitOps repository and push the changes:

```bash
git add helm-release.yaml
git commit -m "Add HelmRelease for my-app"
git push origin main
```

### 5. Verify Deployment

FluxCD will automatically detect the new HelmRelease manifest and deploy the application using the Helm chart with the specified configuration.

You can verify the deployment by checking the status of the HelmRelease:

```bash
flux get helmreleases -A
```

## Updating the Application

To update the application, modify the `values.yaml` file or the `helm-release.yaml` manifest and push the changes to your GitOps repository. FluxCD will automatically apply the updates.