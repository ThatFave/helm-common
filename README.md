# Helm Common Chart

A flexible Helm chart for deploying arbitrary Docker/OCI images with common configurations.

## Description

This Helm chart provides a generic way to deploy containers in Kubernetes. It supports multiple containers, configurable resources, service exposure, ingress configuration, and more.

## Features

- Multiple container support
- Configurable resource requests and limits
- Service exposure (ClusterIP, NodePort, LoadBalancer)
- Ingress configuration
- Autoscaling (HPA)
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
| `namespace.create` | Create a new namespace for the deployment | `true` |
| `namespace.name` | Name of the namespace to use | `"default"` |
| `containers` | List of containers to deploy | `[{"name": "main", "image": {"repository": "nginx"}}]` |
| `serviceAccount.create` | Create a service account for the deployment | `true` |
| `service.type` | Type of service to create (ClusterIP, NodePort, LoadBalancer) | `"ClusterIP"` |
| `service.port` | Port for the service | `80` |
| `ingress.enabled` | Enable ingress configuration | `false` |

For a full list of configurable parameters and examples, see `values-sample.yaml`.

## Using with FluxCD

To use this Helm chart with FluxCD, you need to create a GitOps repository with the necessary manifests. Here's an example setup:

1. Create a new directory in your GitOps repository for the Helm release:

   ```bash
   mkdir -p clusters/my-cluster/apps/my-app
   ```

2. Create a `values.yaml` file with your custom configuration:

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

3. Create a `helm-release.yaml` file with the following content:

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

4. Create a Kubernetes secret with your `values.yaml` file:

   ```bash
   kubectl create secret generic my-app-values --from-file=values.yaml -n flux-system
   ```

5. Commit and push the changes to your GitOps repository.

FluxCD will automatically detect the new HelmRelease manifest and deploy the application using the Helm chart with the specified configuration.