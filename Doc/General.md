###### Get all kubernetes kind(Object): list
```shell
kubectl api-resources
```

###### K8 create namspace
```shell
kubectl create namespace ilan-dev
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

