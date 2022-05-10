# Task 3

## We published minio "outside" using nodePort. Do the same but with ingress.

Apply configs
```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl apply -f deployment.yaml
kubectl apply -f minio-service.yaml
kubectl apply -f ingress-outside.yaml
```

Trying to connect to minikube
```bash
curl -I -k http://192.168.59.105/
HTTP/1.1 200 OK
Date: Tue, 10 May 2022 13:17:58 GMT
Content-Type: text/html
Content-Length: 1356
Connection: keep-alive
Accept-Ranges: bytes
Last-Modified: Sat, 29 Jan 2022 12:35:19 GMT
Vary: Accept-Encoding
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-Xss-Protection: 1; mode=block
```

### Publish minio via ingress so that minio by ip_minikube and nginx returning hostname (previous job) by path ip_minikube/web are available at the same time.

Delete priveous ingress and apply new config
```bash
kubectl delete -f ingress-outside.yaml
kubectl apply -f ingress.yaml
```

Trying to connect ip_minikube and we can see redirect ip_minikube/web
```bash
curl -I -k http://192.168.59.105/
HTTP/1.1 302 Moved Temporarily
Date: Tue, 10 May 2022 13:25:32 GMT
Content-Type: text/html
Content-Length: 138
Connection: keep-alive
Location: http://192.168.59.105/web
```

Trying to connect ip_minikube/web
```bash
curl -I -k http://192.168.59.100/web
HTTP/1.1 200 OK
Date: Tue, 10 May 2022 13:26:12 GMT
Content-Type: text/html
Content-Length: 1356
Connection: keep-alive
Accept-Ranges: bytes
Last-Modified: Tue, 10 May 2022 13:26:12 GMT
Vary: Accept-Encoding
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-Xss-Protection: 1; mode=block
```

### Create deploy with emptyDir save data to mountPoint emptyDir, delete pods, check data.

Apply config
```bash
kubectl apply -f deployment-empty_dir.yaml
```

Create file in volume
```bash
kubectl get po
NAME                     READY   STATUS    RESTARTS   AGE
minio-26f74574fs-mkr26   1/1     Running   0          7s

kubectl exec -it minio-26f74574fs-mkr26 -- bash
touch /data/test_file
```

Check volume
```bash
ls -la /data/
total 12
drwxrwxrwx 3 root root 4096 May 10 13:31 .
drwxr-xr-x 1 root root 4096 May 10 13:30 ..
drwxr-xr-x 6 root root 4096 May 10 13:30 .minio.sys
-rw-r--r-- 1 root root    0 May 10 13:31 test_file
```

Delete pod
```bash
kubectl delete pod minio-69f75574bd-mkr26
pod "minio-26f74574fs-mkr26" deleted
```

Get pod name
```bash
kubectl get po
NAME                     READY   STATUS    RESTARTS   AGE
minio-26f74574fs-wmb29   1/1     Running   0          63s
```

Connect new pod 
```bash
kubectl exec -it minio-26f74574fs-wmb29 -- bash 
```

See that "test_file" file also be deleted
```bash
ls -la /data/
total 12
drwxrwxrwx 3 root root 4096 May 10 13:33 .
drwxr-xr-x 1 root root 4096 May 10 13:33 ..
drwxr-xr-x 6 root root 4096 May 10 13:33 .minio.sys
```