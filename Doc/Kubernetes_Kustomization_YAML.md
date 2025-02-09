Here’s a basic example of Kubernetes Kustomization YAML files.

### 1. Directory Structure

```tree
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

### 2. Base Kustomization

base/deployment.yaml
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

base/service.yaml

```yaml

```