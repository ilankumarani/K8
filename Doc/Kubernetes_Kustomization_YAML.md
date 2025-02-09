# Kubernetes Kustomization Example

## Overview
This repository demonstrates how to use **Kustomize** to manage Kubernetes configurations for different environments (Dev and Prod).

## Directory Structure
```
kustomize-example/
│── base/
│   │── deployment.yaml
│   │── service.yaml
│   │── kustomization.yaml
│── overlays/
│   ├── dev/
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
        ports:
        - containerPort: 80
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
      targetPort: 80
```

### `base/kustomization.yaml`
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
```

## Overlays for Different Environments
Each environment has a separate overlay that modifies the base configuration.

### Dev Environment

#### `overlays/dev/kustomization.yaml`
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base
patchesStrategicMerge:
  - patch-deployment.yaml
```

#### `overlays/dev/patch-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:dev
```

### Prod Environment

#### `overlays/prod/kustomization.yaml`
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base
patchesStrategicMerge:
  - patch-deployment.yaml
```

#### `overlays/prod/patch-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 5
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:prod
```

## Applying Kustomization
Use `kubectl apply -k` to deploy different environments:

### Deploy Dev Environment:
```sh
kubectl apply -k overlays/dev
```

### Deploy Prod Environment:
```sh
kubectl apply -k overlays/prod
```

## Conclusion
This setup allows you to maintain a single base configuration while customizing it for different environments using **Kustomize**. 🚀

