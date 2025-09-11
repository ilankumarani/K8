## Namespace

A namespace in Kubernetes (k8s) is like a folder that helps organize and separate different groups of resources (like pods, services, deployments) inside the same cluster.

**Think of it like:**

*	Your computer has one big hard drive (the cluster).
*	You create different folders (the namespaces) to keep files for different projects separate, even though they’re all on the same hard drive.

[Click to learn](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

-----

## POD and POD lifecycle

[Click to learn about Pod](Doc/Pod.md)

------
## Request

------
## Limit


------
## ReplicaSet

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
> **If you delete the ReplicaSet all pods will be deleted.**
>
> [Click here to learn](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)


------
## DemonSet
A DaemonSet ensures that all (or some) Nodes run a copy of a Pod(example Splunk Agent in each Node)

[Click to know more about DemonSet](Doc/readMe/DaemonSet-README.md)

------
## K8 commands
[Click to know more about Commands](Doc/General.md)