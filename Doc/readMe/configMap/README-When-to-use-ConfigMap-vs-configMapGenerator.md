# README — When to use Kubernetes **ConfigMap** vs Kustomize **configMapGenerator**

A quick, practical guide to choosing between a plain **ConfigMap** and Kustomize **configMapGenerator**, with minimal examples and rollout tips.

---

## Simple words
- **Use a plain `ConfigMap` manifest** when you just need a fixed config file/keys and you’re okay managing the YAML by hand.
- **Use Kustomize `configMapGenerator`** when your config comes from local files/literals/env files and you want **automatic name hashing** so Deployments roll on config changes, plus easy **dev/prod overlays**.

---

## Technical explanation

### Decision cheat‑sheet
**Pick a plain `ConfigMap` when…**
- You don’t use Kustomize (e.g., raw `kubectl apply`, or Helm-only workflow).
- You need a **stable name** (e.g., another system hardcodes `name: app-config` and cannot follow hashed names).
- You want fields the generator doesn’t expose directly (e.g., `immutable: true` without patching).
- The config is tiny and rarely changes; you prefer explicit YAML.

**Pick `configMapGenerator` when…**
- You use **Kustomize/GitOps** and want the name **hash** to change automatically on content changes → triggers rollouts safely.
- You want **overlays** (dev/stage/prod) to merge/replace values cleanly.
- You source values from **files, literals, or `.env` files** and don’t want to hand-embed them in YAML.
- You want Kustomize to **rewrite references** (envFrom/configMapRef/volumes) to the hashed name for you.

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
**Pros:** simple, stable name, supports `immutable: true`.
**Cons:** no auto-hash → you must force rollouts yourself (e.g., checksum annotation or `kubectl rollout restart`).

### 2) Kustomize `configMapGenerator` (auto-hash + overlays)

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
  # default is false (hash enabled) — keep it for auto-rollouts
  disableNameSuffixHash: false
```

**`base/deployment.yaml`** (reference base name; Kustomize rewrites to hashed name)
```yaml
envFrom:
  - configMapRef:
      name: app-config
...
volumes:
  - name: cfg
    configMap:
      name: app-config
```

**`overlays/prod/kustomization.yaml`**
```yaml
resources:
  - ../../base
configMapGenerator:
  - name: app-config
    behavior: merge
    literals:
      - LOG_LEVEL=warn
```

**Pros:** name changes on content → Deployments roll; easy per-env differences.
**Cons:** name is **not stable** (has suffix). If a consumer needs a fixed name, either:
- set `disableNameSuffixHash: true` (you must handle rollouts), or
- keep hash on and add an extra **Service/Env indirection** or **patch** consumers.

---

## Common gotchas & tips
- **Rollouts:** Plain `ConfigMap` won’t restart Pods on change. Use a **checksum annotation** on the Pod template or `kubectl rollout restart`. Generators solve this by changing the name (hash).
- **Immutable configs:** If you want `immutable: true`, either use a plain `ConfigMap`, or **patch the generated** resource:
  ```yaml
  patchesStrategicMerge:
    - cm-immutable-patch.yaml
  ```
  `cm-immutable-patch.yaml`:
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-config
  immutable: true
  ```
- **Stable name required?** Set `disableNameSuffixHash: true`, but then add your own rollout trigger.
- **SubPath caveat:** Whether generated or not, `subPath` mounts **do not auto-update**; mount the **directory** to get live file updates.
- **Binary data:** ConfigMaps are text only. For binaries, use a Secret or bake files into the image/volume.

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
