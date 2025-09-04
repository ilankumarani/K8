## What is POD ?

**Simple words:**

A Pod is like a small box that holds one or more containers.

- The containers in the box share the same address and storage.
- Kubernetes starts, stops, and moves the whole box as one thing.

**Technical explanation:**

A Pod is Kubernetes’ smallest deployable unit: one or more containers that run together on the same node, sharing the IP/ports, localhost network, and optional volumes. They’re scheduled, scaled, and restarted as a single unit.

---

## Lifecycle of pod
[Click here to learn about Lifecycle](PDF/LifeCycle_of_POD.pdf)


> [!NOTE]
>
> CrashLoopBackOff is the one important lifecycle of POD
> 
> [Click here to learn about CrashLoopBackOff](readMe/CrashLoopBackOff-README-section.md)

---