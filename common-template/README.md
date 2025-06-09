# Common Template Helm Chart

This is a common template Helm chart that can be used as a starting point for Kubernetes deployments.

## Features

- Supports regular environment variables
- Supports external secrets as environment variables
- Supports persistent storage configuration

## Usage

### Regular Environment Variables

```yaml
containers:
  - name: app
    image: myapp:latest
    ports:
      - containerPort: 8080
    env:
      - name: APP_ENV
        value: production
```

### External Secrets

To use external secrets as environment variables, define them in the `externalSecrets` section:

```yaml
containers:
  - name: app
    image: myapp:latest
    ports:
      - containerPort: 8080
    env:
      - name: APP_ENV
        value: production
    # Define external secrets that will be used as environment variables
    externalSecrets:
      DB_PASSWORD: my-database-secret
      API_KEY: my-api-key-secret
```

In this example, the `DB_PASSWORD` and `API_KEY` environment variables will be populated from the corresponding Kubernetes secrets.

### Persistent Storage

To enable persistent storage, configure the `persistence` section in your values.yaml:

```yaml
persistence:
  enabled: true
  accessMode: ReadWriteOnce
  size: 10Gi
  existingClaim: "" # Use this if you want to use an existing PVC
  mountPath: /data # Path where the volume will be mounted inside containers
```

If `existingClaim` is not provided, a new PersistentVolumeClaim will be created with the specified size and access mode.

## Example

See [examples/external-secrets-example.yaml](examples/external-secrets-example.yaml) for a complete example.