###### Get all kubernetes kind(Object): list
```shell
kubectl api-resources # Get all kubernetes kind(Object): list
kubectl create namespace ilan-dev # K8 create namespace
kubectl exec -it <pod-name> /bin/bash # interactive POD and see what is inside pod
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ilan-dev
  labels:
    name: ilan-dev
```

###### K8 get Pods details in wider way
```shell
kubectl get pods --all-namespaces
kubectl get pods
kubectl get pod -n <namespace-name>
kubectl get pods -n <namespace-name>
kubectl describe pod <pod-name>
kubectl describe pod <pod-name> -n <namespace>
```

