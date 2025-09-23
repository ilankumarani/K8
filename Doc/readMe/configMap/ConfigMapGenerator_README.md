# Kustomize ConfigMap Generator — Tiny Sample Repo

This repo shows how to use **Kustomize `configMapGenerator`** with a `base` and `dev`/`prod` overlays. It also includes an **env-file alternative**.

## Structure
```
base/
  kustomization.yaml
  deployment.yaml
  service.yaml
  config/application.yml
overlays/
  dev/kustomization.yaml
  prod/kustomization.yaml
env-alt/
  .env
  kustomization.yaml
```

## What it does
- Generates a **ConfigMap** named `app-config` from a YAML file and literals.
- Kustomize appends a **hash** to the name (e.g., `app-config-9hfhb24m7g`) whenever content changes.
- The Deployment references `app-config` and Kustomize auto-rewrites it to the hashed name so a rollout happens on change.

## Quick start

> You need `kubectl` and either `kustomize` or a recent `kubectl` that supports `-k`.

### Dev overlay
```bash
kubectl apply -k overlays/dev
```

### Prod overlay
```bash
kubectl apply -k overlays/prod
```

### (Optional) Use an .env file instead of literals/files
```bash
kubectl apply -k env-alt
```

### Check what will be applied
```bash
kustomize build overlays/dev | less
# or
kubectl kustomize overlays/dev | less
```

### Clean up
```bash
kubectl delete -k overlays/dev
kubectl delete -k overlays/prod
kubectl delete -k env-alt
```

## Notes
- Keep `disableNameSuffixHash` at its default (hash **enabled**) so Pods roll on config changes.
- In overlays, `behavior: merge` overrides/extends ConfigMap data; `replace` replaces the whole data section.
- The example uses `nginx` just to keep the Deployment minimal; swap the image for your app.

---

Happy shipping!
