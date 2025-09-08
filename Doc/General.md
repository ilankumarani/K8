###### Get all kubernetes kind(Object): list
```shell
kubectl api-resources
```

###### K8 create namspace
```shell
kubectl create namespace my-namespace
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ILAN_Dev
  labels:
    name: ILAN_Dev
```

