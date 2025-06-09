# Helm Common Chart

A flexible Helm chart for deploying arbitrary Docker/OCI images with common configurations.

## Description

This Helm chart provides a generic way to deploy containers in Kubernetes. It supports multiple containers, configurable resources, service exposure, and more.

## Features

- Multiple container support
- Configurable resource requests and limits
- Service exposure (ClusterIP, NodePort, LoadBalancer)
- ConfigMaps and Secrets for environment variables
- Volume mounts and persistent storage
- Node affinity, tolerations, and selectors

## Installation

### Prerequisites

- Kubernetes 1.16+
- Helm 3.x

### Installing the Chart

1. Add the repository (if not already added):

   ```bash
   helm repo add helm-common https://example.com/charts
   helm repo update
   ```

2. Install the chart:

   ```bash
   helm install my-release helm-common/helm-common-chart --namespace my-namespace
   ```

3. To install with custom values, create a `values.yaml` file and run:

   ```bash
   helm install my-release helm-common/helm-common-chart -f values.yaml --namespace my-namespace
   ```

## Uninstalling the Chart

To uninstall/delete the release:

```bash
helm uninstall my-release --namespace my-namespace
```

## Configuration

The following table lists the configurable parameters of the Helm chart and their default values.

| Parameter | Description | Default |
| --- | --- | --- |
| `containers` | List of containers to deploy | `[{"name": "main", "image": {"repository": "nginx"}}]` |
| `serviceAccount.create` | Create a service account for the deployment | `true` |
| `service.type` | Type of service to create (ClusterIP, NodePort, LoadBalancer) | `"ClusterIP"` |
| `service.port` | Port for the service | `80` |

For a full list of configurable parameters and examples, see `values-sample.yaml`.

## Using with FluxCD

To use this Helm chart with FluxCD, you need to create a GitOps repository with the necessary manifests. Here's an example setup:

1. Create a new directory in your GitOps repository for the Helm release:

  ```yaml
  apiVersion: source.toolkit.fluxcd.io/v1beta2
  kind: OCIRepository
  metadata:
    name: helm-common
    namespace: flux-system
  spec:
    interval: 1h
    url: oci://ghcr.io/thatfave/helm-common/helm-common-chart
    ref:
      semver: ">= 0.1.0"
  ```

2. Create a `helm-release.yaml` file with the following content:

  ```yaml
  apiVersion: helm.toolkit.fluxcd.io/v2
  kind: HelmRelease
  metadata:
    name: helm-common
    namespace: helm-common
  spec:
    interval: 5m
    chartRef:
      kind: OCIRepository
      name: helm-common
      namespace: flux-system
    values:
      containers:
        - name: web
          image:
            repository: nginx
            tag: stable
          env:
            - name: ENVIRONMENT
              value: production

      service:
        type: ClusterIP
        port: 80
  ```

3. Commit and push the changes to your GitOps repository.

FluxCD will automatically detect the new HelmRelease manifest and deploy the application using the Helm chart with the specified configuration.