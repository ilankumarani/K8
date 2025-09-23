# Kubernetes ConfigMap Examples — README

Here are clean, copy-pasteable **ConfigMap examples** and how to use them.

## 1) ConfigMap (key/values + a small config file)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  APP_NAME: "my-service"
  LOG_LEVEL: "info"
  config.yml: |
    server:
      port: 8080
    features:
      coolStuff: true
```

## 2) Use it as environment variables (Deployment)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: app
          image: nginx:1.27
          # bring in ALL keys as env vars
          envFrom:
            - configMapRef:
                name: app-config
          # or pull ONE key as an env var
          env:
            - name: ONE_LOG_LEVEL
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: LOG_LEVEL
```

## 3) Mount it as files (Deployment)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-files
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp-files
  template:
    metadata:
      labels:
        app: myapp-files
    spec:
      containers:
        - name: app
          image: busybox:1.36
          command: ["sh","-c","ls -R /etc/app && cat /etc/app/config.yml && sleep 3600"]
          volumeMounts:
            - name: app-config-vol
              mountPath: /etc/app
              readOnly: true
            # mount a single key as a specific filename
            - name: app-config-vol
              mountPath: /etc/app/log-level
              subPath: LOG_LEVEL
              readOnly: true
      volumes:
        - name: app-config-vol
          configMap:
            name: app-config
```

## 4) Handy `kubectl` ways to create/inspect
```bash
# from literals
kubectl create configmap app-config   --from-literal=APP_NAME=my-service   --from-literal=LOG_LEVEL=info

# from a file (key becomes filename unless you rename it with =)
kubectl create configmap app-config --from-file=config.yml=./config.yml

# see it
kubectl get cm app-config -o yaml
```

## Notes
- **Size limit** ~1 MiB per ConfigMap.  
- Updating a ConfigMap doesn’t auto-restart Pods. To pick up changes:  
  `kubectl rollout restart deployment/myapp` (or use your CI to bump a checksum annotation).  
- For binary content use `binaryData` instead of `data`.
