# Task 2

## In Minikube in namespace kube-system, there are many different pods running. Your task is to figure out who creates them, and who makes sure they are running (restores them after deletion).

```bash
kubectl get all -n kube-system
NAME                                   READY   STATUS    RESTARTS      AGE
pod/coredns-64897985d-t4qjg            1/1     Running   1 (17h ago)   19h
pod/etcd-minikube                      1/1     Running   1 (17h ago)   19h
pod/kube-apiserver-minikube            1/1     Running   1 (17h ago)   19h
pod/kube-controller-manager-minikube   1/1     Running   1 (17h ago)   19h
pod/kube-proxy-b9gtb                   1/1     Running   1 (17h ago)   19h
pod/kube-scheduler-minikube            1/1     Running   1 (17h ago)   19h
pod/metrics-server-8579cbb685-m4zrl    1/1     Running   1 (17h ago)   18h
pod/storage-provisioner                1/1     Running   2 (13m ago)   19h

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns         ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   19h
service/metrics-server   ClusterIP   10.103.48.188   <none>        443/TCP                  18h

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   19h

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns          1/1     1            1           19h
deployment.apps/metrics-server   1/1     1            1           18h

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-64897985d           1         1         1       19h
replicaset.apps/metrics-server-79c47b44bf   0         0         0       18h
replicaset.apps/metrics-server-847dcc659d   0         0         0       18h
replicaset.apps/metrics-server-8579cbb685   1         1         1       18h
```
Controlled by higher level kubernetes like Deployment, Replicaset, Replication controller.

---

## Implement Canary deployment of an application via Ingress. Traffic to canary deployment should be redirected if you add "canary:always" in the header, otherwise it should go to regular deployment. Set to redirect a percentage of traffic to canary deployment.

### Prod deployment
```bash
kubectl apply -f deployment-prod.yaml
kubectl apply -f service-prod.yaml
kubectl apply -f ingress-prod.yaml
```
### Canary deployment
```bash
kubectl apply -f deployment-canary.yaml
kubectl apply -f service-canary.yaml
kubectl apply -f ingress-canary.yaml
```

### See all
```bash
kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/web-canary-59dbddbb7f-ggg6s   1/1     Running   0          12s
pod/web-canary-59dbddbb7f-k8p88   1/1     Running   0          12s
pod/web-canary-59dbddbb7f-kpvhv   1/1     Running   0          12s
pod/web-prod-bcb5c7b6d-4z582      1/1     Running   0          117s
pod/web-prod-bcb5c7b6d-lpwfj      1/1     Running   0          117s
pod/web-prod-bcb5c7b6d-r7694      1/1     Running   0          117s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   16m
service/web-canary   ClusterIP   10.104.46.240   <none>        80/TCP    12s
service/web-prod     ClusterIP   10.96.248.230   <none>        80/TCP    45s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web-canary   3/3     3            3           12s
deployment.apps/web-prod     3/3     3            3           117s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/web-canary-59dbddbb7f   3         3         3       12s
replicaset.apps/web-prod-bcb5c7b6d      3         3         3       117s
```

### Try connet 10 times to cluster ip via ingress
```bash
for i in $(seq 1 20); do curl -s 192.168.59.104; done
```

### In response mixed traffic:
```bash
web-prod-bcb5c7b6d-r7694
web-prod-bcb5c7b6d-lpwfj
web-prod-bcb5c7b6d-4z582
web-prod-bcb5c7b6d-r7694
web-prod-bcb5c7b6d-lpwfj
web-prod-bcb5c7b6d-4z582
web-prod-bcb5c7b6d-r7694
web-prod-bcb5c7b6d-lpwfj
web-prod-bcb5c7b6d-lpwfj
web-prod-bcb5c7b6d-4z582
web-prod-bcb5c7b6d-r7694
web-prod-bcb5c7b6d-lpwfj
web-prod-bcb5c7b6d-4z582
web-prod-bcb5c7b6d-r7694
web-prod-bcb5c7b6d-4z582
web-prod-bcb5c7b6d-r7694
web-prod-bcb5c7b6d-lpwfj
web-canary-59dbddbb7f-k8p88
web-prod-bcb5c7b6d-lpwfj
web-prod-bcb5c7b6d-4z582
```

### Add header "canary:always" in request 
```bash
for i in $(seq 1 20); do curl -s -H "canary:always" 192.168.59.104; done
```

### In response only canary traffic:
```bash
web-canary-59dbddbb7f-k8p88
web-canary-59dbddbb7f-kpvhv
web-canary-59dbddbb7f-ggg6s
web-canary-59dbddbb7f-ggg6s
web-canary-59dbddbb7f-k8p88
web-canary-59dbddbb7f-kpvhv
web-canary-59dbddbb7f-ggg6s
web-canary-59dbddbb7f-k8p88
web-canary-59dbddbb7f-kpvhv
web-canary-59dbddbb7f-kpvhv
web-canary-59dbddbb7f-ggg6s
web-canary-59dbddbb7f-k8p88
web-canary-59dbddbb7f-kpvhv
web-canary-59dbddbb7f-ggg6s
web-canary-59dbddbb7f-k8p88
web-canary-59dbddbb7f-kpvhv
web-canary-59dbddbb7f-k8p88
web-canary-59dbddbb7f-ggg6s
web-canary-59dbddbb7f-ggg6s
web-canary-59dbddbb7f-k8p88
```