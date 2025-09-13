# Kubernetes Example – Deployments, Service, and NetworkPolicy

This example demonstrates:  
1. Two **Deployments** (frontend and backend) with different labels.  
2. One **Service** that exposes both frontend and backend Pods using a **common label**.  
3. A **NetworkPolicy** that applies to Pods using the `IN` operator to match multiple labels.

---

## 📦 1. Frontend Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: front-end-app
      project: food-app
  template:
    metadata:
      labels:
        app: front-end-app
        project: food-app   # shared label
    spec:
      containers:
      - name: frontend-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

✅ Runs 2 replicas of Nginx.  
✅ Labels: `app=front-end-app`, `project=food-app`.  
✅ The **shared label `project=food-app`** is used by the Service to group Pods together.

---

## 📦 2. Backend Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: back-end-app
      project: food-app
  template:
    metadata:
      labels:
        app: back-end-app
        project: food-app   # shared label
    spec:
      containers:
      - name: backend-container
        image: redis:latest
        ports:
        - containerPort: 6379
```

✅ Runs 2 replicas of Redis.  
✅ Labels: `app=back-end-app`, `project=food-app`.  
✅ Shares the same `project=food-app` label with the frontend Deployment.

---

## 🌐 3. Service – Exposes Both Deployments

```yaml
apiVersion: v1
kind: Service
metadata:
  name: food-service
spec:
  selector:
    project: food-app   # 👈 common label on both frontend & backend Pods
  ports:
    - name: http
      port: 80
      targetPort: 80
    - name: redis
      port: 6379
      targetPort: 6379
  type: ClusterIP
```

✅ Uses the **common label** `project=food-app` to match both frontend and backend Pods.  
✅ Exposes:  
- Port `80` → routes to Nginx containers.  
- Port `6379` → routes to Redis containers.  
✅ Type `ClusterIP` → accessible only inside the cluster.  

---

## 🔒 4. NetworkPolicy – Using `IN` Expression

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-multiple-apps
spec:
  podSelector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - front-end-app
          - back-end-app   # 👈 matches both frontend and backend Pods
  policyTypes:
  - Egress
  egress:
  - to:
      - ipBlock:
          cidr: 8.8.8.8/32   # allow egress to Google DNS
    ports:
      - protocol: TCP
        port: 53
```

✅ Applies to Pods where `app` label is either **front-end-app** or **back-end-app**.  
✅ `operator: In` allows matching multiple label values in a single rule.  
✅ Allows only **egress traffic to 8.8.8.8:53 (Google DNS)**.  
✅ All other outbound traffic will be blocked (if default-deny policy is enforced by your CNI plugin).  

---

## 🟢 Key Takeaways

- **Deployments** define Pods with unique `app` labels and a shared `project` label.  
- **Service** can’t use `IN`, so it relies on the **shared label** (`project=food-app`) to select both Deployments’ Pods.  
- **NetworkPolicy** supports `IN` via `matchExpressions`, allowing one policy to cover multiple apps.  
- Combining these patterns gives you fine control over how Pods are grouped, exposed, and restricted.  
