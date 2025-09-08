# DaemonSet — Quick README

## Simple words
A **DaemonSet** makes sure **one Pod runs on every node** (or on a chosen group of nodes).  
It’s used for **node-wide agents** like logging, monitoring, networking, or storage helpers.

- New node joins → a Pod is started there automatically.  
- Node leaves → its Pod is removed.  

## Technical explanation
A **DaemonSet** controller ensures **exactly one Pod per matching node** using label selectors (`nodeSelector`/`nodeAffinity`) and **tolerations** (so it can run on tainted nodes). It reacts to node add/remove events, often with `hostNetwork`, `hostPID`, or `hostPath` to access node resources. Updates are **rolling**; you don’t set `replicas` (count comes from nodes). Typical workloads: **Fluent Bit/Vector (logs)**, **node-exporter (metrics)**, **CNI/CSI agents**.

---

## Minimal YAML example
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-agent
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: node-agent
  template:
    metadata:
      labels:
        app: node-agent
    spec:
      tolerations:
        - key: "node-role.kubernetes.io/master"
          effect: "NoSchedule"     # allow on control-plane if needed
          operator: "Exists"
      nodeSelector:
        nodepool: workers           # run only on worker nodes (example)
      containers:
        - name: agent
          image: ghcr.io/your-org/agent:1.0.0
          securityContext:
            allowPrivilegeEscalation: false
          resources:
            requests:
              cpu: 50m
              memory: 64Mi
          volumeMounts:
            - name: varlog
              mountPath: /var/log
              readOnly: true
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
            type: Directory
```

---

## Common uses
- **Logs:** Fluent Bit/Vector tailing `/var/log/*`  
- **Metrics:** node-exporter, kube-state-metrics (note: KSM is a Deployment)  
- **Networking:** CNI daemons (Calico, Cilium)  
- **Storage:** CSI node plugins

---

## Quick commands
```bash
kubectl get ds -A
kubectl describe ds node-agent -n kube-system
kubectl get pods -l app=node-agent -n kube-system -o wide
```

## Troubleshooting tips
- **No pods on some nodes?** Check `nodeSelector`/`affinity` and **taints/tolerations**.  
- **Mount errors?** Verify `hostPath` exists and permissions are correct.  
- **Needs node access?** Consider `hostNetwork: true` or required capabilities carefully.
