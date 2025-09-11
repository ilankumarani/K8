## 1) Namespace

A namespace in Kubernetes (k8s) is like a folder that helps organize and separate different groups of resources (like pods, services, deployments) inside the same cluster.

**Think of it like:**

*	Your computer has one big hard drive (the cluster).
*	You create different folders (the namespaces) to keep files for different projects separate, even though they’re all on the same hard drive.

[Click to learn](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

-----

## 2) POD and POD lifecycle

[Click to learn about Pod](Doc/Pod.md)

------
## 3) Request & Limit

**Request →** the minimum amount of CPU/Memory a Pod needs to run. Kubernetes will guarantee at least this much.

**Limit →** the maximum amount of CPU/Memory a Pod can use. Kubernetes won’t let it go beyond this.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: mycontainer
      image: nginx
      resources:
        requests:
          cpu: "250m"     # minimum 0.25 CPU
          memory: "256Mi" # minimum 256 MB
        limits:
          cpu: "500m"     # maximum 0.5 CPU
          memory: "512Mi" # maximum 512 MB
```
### 3.1) LimitRange and ResourceQuota
[Read more about yaml](Doc/readMe/K8s-Resource-Management-Full-README.md)

**How it works with LimitRange:**
*	**LimitRange →** sets defaults per Pod/container.
*	**ResourceQuota →** sets the overall ceiling for the Namespace.

**Think of it like:**
*	**LimitRange =** rules for each employee (Pod).
*	**ResourceQuota =** total company budget (Namespace).

------
## 4) ReplicaSet

A Replica in Kubernetes means how many copies of your Pod you want running at the same time.

**Example:**

*	If you set replicas = 3, Kubernetes makes sure 3 identical Pods are always running.
*	If one Pod crashes, Kubernetes will automatically start a new one so that the count always stays at 3.

**Get replicaset:**
```shell
kubectl get rs
```

```shell
kubectl describe rs/<replicaSet_name>
```

**Delete replicaset:**
```shell
kubectl get rs <replicaSet_name>
```

> [!NOTE]
>
> **Deleting a ReplicaSet also deletes all its Pods associated with that.**
>
> [Click here to learn](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)


------
## 5) DemonSet
A DaemonSet ensures that all (or some) Nodes run a copy of a Pod(example Splunk Agent in each Node)

[Click to know more about DemonSet](Doc/readMe/DaemonSet-README.md)

------
## K8 commands
[Cheat sheet](https://spacelift.io/blog/kubernetes-cheat-sheet)

[Click to know more about Commands](Doc/General.md)