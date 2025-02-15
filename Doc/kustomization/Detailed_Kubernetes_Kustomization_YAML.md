# Kubernetes Kustomization - Detailed Example

## Overview
This repository demonstrates an advanced **Kustomize** setup for managing Kubernetes configurations across multiple environments (`dev`, `staging`, and `prod`).

## Directory Structure
```
kustomize-example/
│── base/
│   │── deployment.yaml
│   │── service.yaml
│   │── configmap.yaml
│   │── secret.yaml
│   │── kustomization.yaml
│── overlays/
│   ├── dev/
│   │   │── kustomization.yaml
│   │   │── patch-deployment.yaml
│   ├── staging/
│   │   │── kustomization.yaml
│   │   │── patch-deployment.yaml
│   ├── prod/
│   │   │── kustomization.yaml
│   │   │── patch-deployment.yaml
```

## Base Configuration
The **base** directory contains the common Kubernetes configurations.

### `base/deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:latest
        envFrom:
        - configMapRef:
            name: my-app-config
        - secretRef:
            name: my-app-secret
        ports:
        - containerPort: 8080
```

### `base/service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

### `base/configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-app-config
data:
  APP_ENV: "default"
  LOG_LEVEL: "info"
```

### `base/secret.yaml`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-app-secret
type: Opaque
stringData:
  API_KEY: "default-secret"
```

### `base/kustomization.yaml`
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
  - configmap.yaml
  - secret.yaml

configMapGenerator:
  - name: my-app-config
    literals:
      - APP_ENV=default
      - LOG_LEVEL=info

secretGenerator:
  - name: my-app-secret
    literals:
      - API_KEY=default-secret

commonLabels:
  project: my-app
```

## Overlays
Each environment has a separate overlay that modifies the base configuration.

### Development Overlay
#### `overlays/dev/kustomization.yaml`
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: dev
resources:
  - ../../base

patchesStrategicMerge:
  - patch-deployment.yaml

configMapGenerator:
  - name: my-app-config
    literals:
      - APP_ENV=development
      - LOG_LEVEL=debug

secretGenerator:
  - name: my-app-secret
    literals:
      - API_KEY=dev-secret-key

images:
  - name: my-app
    newTag: dev-latest
```

### Staging Overlay
#### `overlays/staging/kustomization.yaml`
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: staging
resources:
  - ../../base

patchesStrategicMerge:
  - patch-deployment.yaml

configMapGenerator:
  - name: my-app-config
    literals:
      - APP_ENV=staging
      - LOG_LEVEL=warn

secretGenerator:
  - name: my-app-secret
    literals:
      - API_KEY=staging-secret-key

images:
  - name: my-app
    newTag: staging-latest
```

### Production Overlay
#### `overlays/prod/kustomization.yaml`
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: prod
resources:
  - ../../base

patchesStrategicMerge:
  - patch-deployment.yaml

configMapGenerator:
  - name: my-app-config
    literals:
      - APP_ENV=production
      - LOG_LEVEL=error

secretGenerator:
  - name: my-app-secret
    literals:
      - API_KEY=prod-secret-key

images:
  - name: my-app
    newTag: prod-latest
```

## Applying Kustomization
To deploy specific environments, use the following commands:

### Deploy Dev Environment:
```sh
kubectl apply -k overlays/dev
```

### Deploy Staging Environment:
```sh
kubectl apply -k overlays/staging
```

### Deploy Production Environment:
```sh
kubectl apply -k overlays/prod
```

## Key Features Used
✅ **Namespace Overlays** (`namespace: dev/staging/prod`)

✅ **Patching Deployments** (`patchesStrategicMerge`)

✅ **ConfigMap & Secret Generators** (`configMapGenerator` & `secretGenerator`)

✅ **Image Overrides** (`images:` section for updating tags)

✅ **Common Labels** (`commonLabels:` section for organization)

This setup ensures **scalability, environment isolation, and easy management** for Kubernetes applications! 🚀

