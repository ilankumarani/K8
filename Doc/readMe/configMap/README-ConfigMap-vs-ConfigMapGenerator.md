# Choosing Between Kubernetes **ConfigMap** vs Kustomize **configMapGenerator**
A practical, copy‑pasteable README that helps you decide **when to use which**, with minimal examples and rollout patterns.

---

## Simple words
- **Use a plain `ConfigMap`** when you just need a fixed YAML you’ll manage by hand (or you’re not using Kustomize/overlays).

- **Use Kustomize `configMapGenerator`** when your config comes from files/literals/env files and you want **automatic name hashing** so Deployments **roll on changes**, plus easy **dev/prod overlays**.

---

## Technical explanation (short & focused)

### When to pick a plain `ConfigMap`

- You **don’t** use Kustomize (raw `kubectl apply` or Helm‑only).

- You need a **stable name** (some external system hardcodes `name: app-config`).

- You want fields the generator doesn’t expose directly (e.g., `immutable: true` without extra patches).

- The config is tiny/rarely changes; you prefer explicit YAML and manual rollouts.



**Pros:** simple, stable, supports `immutable: true` out of the box.

**Cons:** **no auto rollout** on content change; you must trigger it (checksum annotation or `kubectl rollout restart`).



### When to pick `configMapGenerator`

- You use **Kustomize/GitOps** and want the **name hash** to change when content changes → pods roll safely.

- You want **overlays** (dev/stage/prod) that merge/replace values cleanly.

- You source values from **files**, **literals**, or **`.env`** files and don’t want to hand‑embed them in YAML.

- You want Kustomize to **rewrite all references** (envFrom/volume configMapRef) to the new hashed name for you.



**Pros:** automatic rollout via hash; great for per‑env config.

**Cons:** the resource name is **not stable** (hash suffix). If a consumer needs a fixed name, you must disable hashing or add an indirection/patch.



---

## Minimal examples

### 1) Plain `ConfigMap` (manual)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config              # stable name
  namespace: default
data:
  LOG_LEVEL: "info"
  application.yml: |
    server:
      port: 8080
```

**Rollout trigger (recommended):** add a checksum annotation on the Pod template so changes trigger a rollout:

```yaml
# deployment.yaml (fragment)
spec:
  template:
    metadata:
      annotations:
        config.checksum/app-config: "<sha256 of the ConfigMap content>"
```

Compute the checksum in CI/Helm/Kustomize and inject it.



### 2) Kustomize `configMapGenerator` (auto‑hash + overlays)

**`base/kustomization.yaml`**

```yaml
resources:
  - deployment.yaml

configMapGenerator:
  - name: app-config
    files:
      - application.yml=config/application.yml
    literals:
      - LOG_LEVEL=info

generatorOptions:
  # default behavior adds a hash suffix based on content
  disableNameSuffixHash: false
```

**`base/deployment.yaml`** (reference the *base* name; Kustomize rewrites to the hashed name)

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
                name: app-config           # Kustomize rewrites to app-config-<hash>
          volumeMounts:
            - name: cfg
              mountPath: /config
              readOnly: true
      volumes:
        - name: cfg
          configMap:
            name: app-config               # also rewritten
```

**Overlays**

`overlays/dev/kustomization.yaml`

```yaml
resources:
  - ../../base

configMapGenerator:
  - name: app-config
    behavior: merge
    literals:
      - LOG_LEVEL=debug
```

`overlays/prod/kustomization.yaml`

```yaml
resources:
  - ../../base

configMapGenerator:
  - name: app-config
    behavior: merge
    literals:
      - LOG_LEVEL=warn
```

Build/apply:

```bash
kubectl apply -k overlays/dev   # or overlays/prod
```

---

## Patterns & tips

- **Stable name required?** Set `generatorOptions.disableNameSuffixHash: true` **or** keep the hash and provide an indirection (e.g., refer via an env var/patch), but remember to add your **own rollout trigger**.

- **Immutable configs:** Plain `ConfigMap` supports `immutable: true`. With Kustomize, patch the generated resource:

```yaml
# patchesStrategicMerge: [cm-immutable.yaml]
# cm-immutable.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
immutable: true
```

- **Avoid `subPath`** for dynamic reloads. Mount the **directory** so the projected files update in place. `subPath` mounts don’t reflect updates; do a rollout instead.

- **Binary data** belongs in a **Secret** or a baked volume/image; ConfigMaps are for text.

- **GitOps**: store source files (`config/application.yml`, `.env`) in repo; let Kustomize produce the CM. The name hash safely drives rollouts.



---

## FAQ

**Q: Does a plain `ConfigMap` restart pods automatically?**  
A: No. Use a **checksum annotation** or `kubectl rollout restart`.

**Q: Will `configMapGenerator` update references automatically?**  
A: Yes. Kustomize rewrites `configMapRef` and `volumes[*].configMap.name` to the hashed name.

**Q: Can I still make it `immutable` with generator?**  
A: Yes—patch the generated resource (see above).

**Q: Which should I use with Helm?**  
A: Helm users typically template a plain `ConfigMap` **plus** a checksum annotation in the Deployment.



---

## Commands cheat‑sheet

```bash
# Plain ConfigMap

kubectl apply -f configmap.yaml

# Force a rollout when the ConfigMap changed but pods didn’t restart

kubectl rollout restart deployment/myapp



# Kustomize builds

kubectl apply -k overlays/dev

kubectl kustomize overlays/prod | less

```

---

## TL;DR decision

- **No Kustomize / need stable name / want immutable** → **Plain ConfigMap** + checksum annotation.

- **Use Kustomize / want auto-rollouts & overlays** → **configMapGenerator** (hash on), reference `app-config` in manifests, let Kustomize rewrite the hashed name.