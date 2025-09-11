# Kubernetes: Requests, Limits, and ResourceQuota

## 1. Requests and Limits (per Pod)
- **Request** → minimum CPU/Memory a Pod is guaranteed.
- **Limit** → maximum CPU/Memory a Pod can use.

### Example Pod YAML with Requests and Limits
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-requests-limits
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        cpu: "250m"
        memory: "256Mi"
      limits:
        cpu: "500m"
        memory: "512Mi"
```

---

## 2. LimitRange (defaults for all Pods in a Namespace)
Use this if you don’t want to repeat requests/limits in every Pod.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: my-namespace
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "250m"
      memory: "256Mi"
    type: Container
```

👉 If a Pod doesn’t specify resources, these defaults will be applied.

---

## 3. ResourceQuota (budget for the Namespace)
Sets the maximum total resources allowed in a Namespace.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ns-budget
  namespace: my-namespace
spec:
  hard:
    requests.cpu: "2"
    requests.memory: "4Gi"
    limits.cpu: "4"
    limits.memory: "8Gi"
    pods: "10"
```

👉 Meaning:
- Max 10 Pods in this namespace.
- All Pods together cannot request more than 2 CPU or 4 GB RAM.
- They cannot consume more than 4 CPU or 8 GB RAM.

---

## Summary
- **Requests/Limits** → per Pod/container control.
- **LimitRange** → default requests/limits for a Namespace.
- **ResourceQuota** → overall budget for a Namespace.

Together, these help manage resources fairly and avoid overloading the cluster.
