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

