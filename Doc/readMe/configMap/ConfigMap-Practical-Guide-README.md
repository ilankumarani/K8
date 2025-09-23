# Kubernetes ConfigMaps — Practical Guide & README

A concise, practical README you can drop into your repo to explain **what ConfigMaps are**, **how to use them**, and **how to operate them in production**.

---

## What is a ConfigMap?
A **ConfigMap** holds non‑secret configuration as **key/values** or **text files**. Pods can consume ConfigMaps as environment variables or mounted files. ConfigMaps are **namespaced** and intended for data you’re comfortable storing in plain text (for secrets, use **Secrets**).

**Hard limits**: keep a single ConfigMap under ~**1 MiB** total payload. Very large configs belong in object storage or a volume.

---

## Quick Start

**ConfigMap**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
  labels:
    app: myapp
    tier: backend
# string data
data:
  APP_NAME: "my-service"
  LOG_LEVEL: "info"
  config.yml: |
    server:
      port: 8080
    features:
      coolStuff: true
```

**Use as environment variables**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels: { app: myapp }
  template:
    metadata:
      labels: { app: myapp }
    spec:
      containers:
        - name: app
          image: nginx:1.27
          envFrom:
            - configMapRef:
                name: app-config   # imports APP_NAME, LOG_LEVEL
          env:
            - name: ONE_LOG_LEVEL # import one key explicitly
              valueFrom:
                configMapKeyRef: { name: app-config, key: LOG_LEVEL }
```

**Use as mounted files**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-files
spec:
  replicas: 1
  selector:
    matchLabels: { app: myapp-files }
  template:
    metadata:
      labels: { app: myapp-files }
    spec:
      containers:
        - name: app
          image: busybox:1.36
          command: ["sh","-c","ls -R /etc/app && cat /etc/app/config.yml && sleep 3600"]
          volumeMounts:
            - name: app-config-vol
              mountPath: /etc/app
              readOnly: true
            # mount a single key to a specific filename
            - name: app-config-vol
              mountPath: /etc/app/log-level
              subPath: LOG_LEVEL
              readOnly: true
      volumes:
        - name: app-config-vol
          configMap:
            name: app-config
            # optional: map key -> filename
            items:
              - key: config.yml
                path: config.yml
            # optional: file permission bits (octal)
            defaultMode: 0644
```

**kubectl helpers**
```bash
# from literals
kubectl create configmap app-config   --from-literal=APP_NAME=my-service   --from-literal=LOG_LEVEL=info

# from file(s)
kubectl create configmap app-config --from-file=config.yml=./config.yml

# inspect
a kubectl get cm app-config -o yaml
```

---

## When to use ConfigMaps vs Secrets
- **ConfigMap**: feature flags, tuning params, non-sensitive endpoints, template files.
- **Secret**: passwords, tokens, private keys, anything confidential. (Base64 in Secret is **not** encryption; enable at‑rest encryption for etcd.)

---

## Patterns & Best Practices

### 1) Immutable ConfigMaps (avoid accidental edits)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
immutable: true
# ...
```
Using `immutable: true` prevents runtime changes; to update, create a new CM with a new name. Good for stability and API server performance.

### 2) Rolling out changes on updates
Updating a ConfigMap **does not** automatically restart Pods. Common strategies:

- **Manual restart**:
  ```bash
  kubectl rollout restart deployment/myapp
  ```
- **Checksum annotation pattern** (forces rollout when CM changes):
  ```yaml
  spec:
    template:
      metadata:
        annotations:
          config.checksum/app-config: "<sha256 of the ConfigMap yaml>"
  ```
  Compute the checksum in CI/Helm/Kustomize and inject it.

- **Auto-reload from file**: if your app can reload on file change, mount CM as files (not `subPath`) so kubelet updates the projected files in-place. Note: `subPath` mounts **do not** update on CM changes.

- **Sidecar reloader**: run a lightweight watcher (e.g., reloader container) that sends SIGHUP/HTTP reload to your app when files change.

### 3) Avoid `subPath` for dynamic config
`subPath` creates a bind mount to a single file at container start. It **won’t** reflect later CM updates. Prefer mounting the whole directory. If you must use `subPath`, trigger a rollout on changes.

### 4) Binary content
For non-UTF8 data use `binaryData` (base64-encoded) and mount as files. Example:
```yaml
apiVersion: v1
kind: ConfigMap
metadata: { name: binary-config }
binaryData:
  cert.der: MIIByDCCAXCgAwIBAgI...  # base64
```

### 5) File permissions & ownership
- Use `defaultMode` to set file mode (e.g., `0440` for read-only).
- To change ownership, rely on Pod `securityContext.fsGroup` so mounted files are group-readable by your app user.

### 6) Names & versioning
Adopt a versioned name pattern to enable parallel deploys:
- `app-config-v2025-09-23-001`
- Reference via label selector or update the Deployment to the new name.

### 7) Keep them small & composable
Break large configs into multiple CMs (e.g., `app-flags`, `app-logging`, `nginx-conf`).

---

## Troubleshooting Checklist
- **Pod won’t start with envFrom**: key not found? set `optional: true` on `configMapRef` while debugging.
- **Changes not visible**: did you use `subPath`? If yes, rollout restart. Otherwise give kubelet ~1min to refresh projected files.
- **Wrong file perms**: set `defaultMode`, verify `fsGroup`.
- **YAML parsing issues**: quote values with `:` or `#`, avoid tabs; use `|` for multi-line.
- **Collision with Secret names/keys**: preferred to separate concerns; if projecting both, use distinct mount paths.

---

## Spring Boot Example (Java)

**ConfigMap**
```yaml
apiVersion: v1
kind: ConfigMap
metadata: { name: spring-config }
data:
  application.yml: |
    server:
      port: 8080
    management:
      endpoints:
        web:
          exposure:
            include: "health,info"
    logging:
      level:
        root: ${LOG_LEVEL:INFO}
  LOG_LEVEL: INFO
```

**Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: spring-app }
spec:
  replicas: 2
  selector: { matchLabels: { app: spring-app } }
  template:
    metadata:
      labels: { app: spring-app }
      annotations:
        config.checksum/spring-config: "{{sha256-of-config}}"  # fill via CI/Helm
    spec:
      containers:
        - name: app
          image: ghcr.io/example/spring-app:1.0.0
          env:
            - name: LOG_LEVEL
              valueFrom:
                configMapKeyRef: { name: spring-config, key: LOG_LEVEL }
          volumeMounts:
            - name: spring-config
              mountPath: /config
              readOnly: true
      volumes:
        - name: spring-config
          configMap:
            name: spring-config
```
Spring Boot (since 2.3+) will pick up `/config/application.yml` automatically if `spring.config.additional-location=/config` or image includes correct search path. Alternatively, set `SPRING_CONFIG_ADDITIONAL_LOCATION=/config/` env.

---

## NGINX Example

**ConfigMap**
```yaml
apiVersion: v1
kind: ConfigMap
metadata: { name: nginx-config }
data:
  default.conf: |
    server {
      listen 8080;
      location /healthz { return 200 'ok'; }
      location / {
        proxy_pass http://127.0.0.1:3000;
      }
    }
```

**Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: nginx }
spec:
  replicas: 1
  selector: { matchLabels: { app: nginx } }
  template:
    metadata:
      labels: { app: nginx }
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 8080
          volumeMounts:
            - name: nginx-config
              mountPath: /etc/nginx/conf.d
              readOnly: true
      volumes:
        - name: nginx-config
          configMap:
            name: nginx-config
            items:
              - key: default.conf
                path: default.conf
```

---

## Helm & Kustomize Snippets

**Helm: ConfigMap from file + checksum annotation**
```yaml
# templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "app.fullname" . }}-config
  labels: {{- include "app.labels" . | nindent 2 }}
data:
  config.yml: |-
    {{- (.Files.Get "config/config.yml") | nindent 4 }}
```
```yaml
# templates/deployment.yaml (fragment)
metadata:
  annotations:
    checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
```

**Kustomize: generate from files**
```yaml
# kustomization.yaml
configMapGenerator:
  - name: app-config
    files:
      - config.yml
options:
  disableNameSuffixHash: false   # enable hash suffix to force rollout on changes
```

---

## Operational Notes
- **RBAC**: reading a CM requires `get` on that resource in the namespace; mounts via volumes are handled by kubelet and don’t require app credentials.
- **Multi-container Pods**: mount the same ConfigMap volume into multiple containers to share config.
- **Performance**: very large projected directories may slow pod start; prune unnecessary keys.
- **Backups**: treat ConfigMaps as part of cluster state—export in Git (GitOps) or include in backups.

---

## FAQ
**Q: Do mounted files update automatically when the ConfigMap changes?**  
A: Yes, for directory mounts of the whole ConfigMap (via projected volumes) after a short delay. **No** for `subPath` mounts.

**Q: Can I reference a ConfigMap across namespaces?**  
A: No—ConfigMaps are namespaced. Copy or template them per namespace.

**Q: How do I keep sensitive values out of ConfigMaps?**  
A: Use Secrets and enable at-rest encryption; mount them separately and reference via env vars or files.

**Q: Can I template values inside a ConfigMap file?**  
A: Use your toolchain (Helm, Kustomize, CI) to render templates before applying.

---

## Command Cheat Sheet
```bash
# create
kubectl apply -f configmap.yaml

# list
kubectl get configmaps

# describe
kubectl describe configmap app-config

# edit (careful in prod)
kubectl edit configmap app-config

# update from local file
kubectl create configmap app-config --from-file=config.yml=./config.yml -o yaml --dry-run=client | kubectl apply -f -

# trigger rollout when config changes
kubectl rollout restart deployment/myapp
```

---

## Appendix: Minimal Example You Can Test Now
```bash
kubectl create ns cm-demo
kubectl -n cm-demo create configmap app-config   --from-literal=APP_NAME=my-service   --from-literal=LOG_LEVEL=info

cat <<'YAML' | kubectl -n cm-demo apply -f -
apiVersion: apps/v1
kind: Deployment
metadata: { name: demo }
spec:
  replicas: 1
  selector: { matchLabels: { app: demo } }
  template:
    metadata: { labels: { app: demo } }
    spec:
      containers:
        - name: demo
          image: busybox:1.36
          command: ["sh","-c","env | grep -E 'APP_NAME|LOG_LEVEL' && sleep 3600"]
          envFrom:
            - configMapRef: { name: app-config }
YAML

kubectl -n cm-demo get pods -w
```

---

**That’s it!** Drop this README into your repo and adapt the examples to your app. If you need a version tuned for Spring Boot/Helm/Kustomize only, copy the relevant section into a separate `README-config.md`.
