# Kubernetes Labels, Selectors, and Services – Example

This example demonstrates how **unique labels** can be applied to different Pods in Kubernetes, and how **Services use selectors** to route traffic only to the Pods that match their label criteria.

---

## 📌 What are Labels and Selectors?

- **Labels** are key-value pairs attached to Kubernetes objects (like Pods) to organize and identify them.
- **Selectors** are used by Services, ReplicaSets, and Deployments to filter and match Pods with specific labels.

---

## 📦 Example Setup

We will create:

1. **Two Pods** – one frontend (`shopping-frontend`) and one backend (`shopping-backend`)  
2. **Two Services** – each Service selects its respective Pod using unique labels  

---

## ✅ YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shopping-frontend-pod
  labels:
    app: shopping-frontend
    tier: ui
spec:
  containers:
  - name: nginx
    image: nginx

---
apiVersion: v1
kind: Pod
metadata:
  name: shopping-backend-pod
  labels:
    app: shopping-backend
    tier: api
spec:
  containers:
  - name: redis
    image: redis

---
apiVersion: v1
kind: Service
metadata:
  name: shopping-frontend-service
spec:
  selector:
    app: shopping-frontend
    tier: ui
  ports:
  - port: 80
    targetPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: shopping-backend-service
spec:
  selector:
    app: shopping-backend
    tier: api
  ports:
  - port: 6379
    targetPort: 6379
```

---

## 🔗 How it Works

- `shopping-frontend-service` selects Pods with labels:
  - `app=shopping-frontend`
  - `tier=ui`  

  → It forwards traffic to **frontend-pod** (running Nginx).

- `shopping-backend-service` selects Pods with labels:
  - `app=shopping-backend`
  - `tier=api`  

  → It forwards traffic to **backend-pod** (running Redis).

---

## 🚀 Key Takeaways

- Use **unique labels** to clearly differentiate Pods in your system.  
- Each Service uses a **selector** to forward traffic only to the matching Pods.  
- This approach helps structure multi-tier applications (frontend, backend, database, etc.).  
