# Task 1

## Create a deployment nginx. Set up two replicas. Remove one of the pods, see what happens.

### Apply deployment
```bash
kubectl apply -f 2-nginx-replicas.yml
```

### Get replicas
```bash
kubectl get po
NAME                                   READY   STATUS    RESTARTS      AGE
two-nginx-deployment-b7fb84cb8-r65rw   1/1     Running   0             5m
two-nginx-deployment-b7fb84cb8-rtng7   1/1     Running   0             5m
```

### Remove one of the pods
```bash
kubectl delete pod two-nginx-deployment-b7fb84cb8-r65rw
pod "two-nginx-deployment-b7fb84cb8-r65rw" deleted
```

### Get replicas
```bash
kubectl get po
NAME                                   READY   STATUS    RESTARTS      AGE
two-nginx-deployment-b7fb84cb8-95xbg   1/1     Running   0             36s
two-nginx-deployment-b7fb84cb8-rtng7   1/1     Running   0             5m
```

Pod with name "two-nginx-deployment-b7fb84cb8-95xbg" have just been created