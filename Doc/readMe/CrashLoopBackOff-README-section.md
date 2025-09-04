## CrashLoopBackOff

### Simple words
A **CrashLoopBackOff** means your container keeps **starting, crashing, and restarting**, and Kubernetes **waits longer each time** before trying again.

- Common causes: bad start command/args, missing config or secrets, wrong image, failing health checks, or required ports not available.

### Technical explanation
`CrashLoopBackOff` indicates the kubelet repeatedly restarts a container that **exits shortly after startup with a non-zero code**. An **exponential backoff** delay is applied between restarts. Triggers include failing entrypoints, runtime exceptions, liveness probe kills, or dependencies not ready.

#### Quick triage
```bash
# See events and back-off reason
kubectl describe pod <pod>

# Inspect current and previous crash logs
kubectl logs <pod> -c <container>
kubectl logs <pod> -c <container> --previous

# Check probes, env, and mounts in the spec
kubectl get pod <pod> -o yaml | less
```