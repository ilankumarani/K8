# Kubernetes Selectors – Complete Guide

Selectors in Kubernetes are used to **filter and match resources** based on labels, fields, or other constraints.  
They are essential for Services, Deployments, NetworkPolicies, scheduling, and more.  

This guide covers:  
1. Pod Selectors  
2. Node Selectors  
3. Label Selectors  
4. Field Selectors  
5. Affinity / Anti-Affinity Selectors  
6. Topology Spread Constraints  
7. Policy Selectors (Namespace / Pod selectors)  

---

## 🟢 Pod Selector
Pod selectors are used to match Pods based on their **labels**.  
They are found in many Kubernetes objects:  
- **Service** → selects Pods to send traffic to  
- **Deployment / ReplicaSet** → manages Pods with specific labels  
- **NetworkPolicy** → applies rules to matching Pods  

### Example: Service with Pod Selector
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:                  # Pod selector
    app: nginx
  ports:
    - port: 80
```

✅ Selects all Pods with `app=nginx`.

---

## 🔵 Node Selector
Node selectors are used in a Pod spec to constrain which **nodes** a Pod can run on.  
They rely on **node labels** (e.g., `disktype=ssd`, `zone=us-east-1a`).  

### Example: Pod with Node Selector
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-selector-pod
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: nginx
    image: nginx
```

✅ Pod will only run on nodes with the label `disktype=ssd`.

---

## 🟣 Label Selectors
Label selectors match resources based on labels, either **equality-based** (`key=value`) or **set-based** (`In`, `NotIn`, `Exists`, `DoesNotExist`).  

### Example: NetworkPolicy (Set-based)
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: select-multiple-apps
spec:
  podSelector:
    matchExpressions:        # Set-based selector
      - key: app
        operator: In
        values:
          - frontend
          - backend
```

✅ Matches Pods with `app=frontend` or `app=backend`.

---

## 🔴 Field Selectors
Field selectors let you select Kubernetes resources based on resource **fields** instead of labels.  

### Examples:
```bash
kubectl get pods --field-selector status.phase=Running
kubectl get pods --field-selector spec.nodeName=node-1
```

✅ Useful for filtering Pods by lifecycle phase or node assignment.  

---

## 🟠 Affinity / Anti-Affinity
Used in **Pod scheduling** for advanced placement rules.  

### Node Affinity
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: disktype
          operator: In
          values:
            - ssd
```

✅ Pod will only schedule on nodes with `disktype=ssd`.

### Pod Anti-Affinity
```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
            - frontend
      topologyKey: "kubernetes.io/hostname"
```

✅ Prevents scheduling Pods with `app=frontend` on the same node.

---

## 🟡 Topology Spread Constraints
Ensure Pods are evenly spread across **nodes or zones**.  

```yaml
topologySpreadConstraints:
- maxSkew: 1
  topologyKey: kubernetes.io/hostname
  whenUnsatisfiable: DoNotSchedule
  labelSelector:
    matchLabels:
      app: demo
```

✅ Ensures Pods with `app=demo` are evenly distributed.

---

## 🟠 Policy Selectors
Used in **NetworkPolicy** or **RBAC** to filter Pods and namespaces.  

### Example: NetworkPolicy with Namespace Selector
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-monitoring
spec:
  podSelector: {}   # all Pods in this namespace
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          purpose: monitoring
```

✅ Only allows traffic from Pods in namespaces labeled `purpose=monitoring`.

---

## ✅ Summary – Types of Selectors

| Selector Type | Where Used | Example |
|---------------|------------|---------|
| **Pod Selector** | Service, Deployment, ReplicaSet, NetworkPolicy | `matchLabels: app=nginx` |
| **Node Selector** | Pod spec, Node Affinity | `nodeSelector: disktype=ssd` |
| **Label Selector** | Almost everywhere | `In`, `NotIn`, `Exists`, `DoesNotExist` |
| **Field Selector** | kubectl, API | `--field-selector status.phase=Running` |
| **Affinity/Anti-Affinity** | Pod scheduling | Node affinity (`ssd`), Pod anti-affinity |
| **Topology Spread** | Deployments | Spread Pods across nodes/zones |
| **Policy Selectors** | NetworkPolicy, RBAC | `namespaceSelector`, `podSelector` |

---

## 🚀 Key Takeaways
- **Pod selector** → Matches Pods using labels (Services, Deployments, NetworkPolicies).  
- **Node selector** → Restricts which nodes Pods run on.  
- **Label selectors** (with `In`, `NotIn`, `Exists`, etc.) → Fine-grained resource matching.  
- **Field selectors** → Filter resources by spec/status fields.  
- **Affinity / Topology selectors** → Control scheduling across nodes/zones.  
- **Policy selectors** → Control traffic and permissions by namespace or Pod.  
