# Task 4

## Create users deploy_view and deploy_edit. Give the user deploy_view rights only to view deployments, pods. Give the user deploy_edit full rights to the objects deployments, pods.

Create private key for users
```bash
openssl genrsa -out deploy_view.key 2048
openssl genrsa -out deploy_edit.key 2048
```
Create a certificate signing request
```bash
openssl req -new -key deploy_view.key -out deploy_view.csr -subj "/CN=deploy_view"
openssl req -new -key deploy_edit.key -out deploy_edit.csr -subj "/CN=deploy_edit"
```
Sign the CSR in the Kubernetes CA (in our case in .minikube).
```bash
openssl x509 -req -in deploy_view.csr -CA C:\Users\mike\.minikube\ca.crt -CAkey C:\Users\mike\.minikube\ca.key -CAcreateserial  -out deploy_view.crt -days 365
openssl x509 -req -in deploy_edit.csr -CA C:\Users\mike\.minikube\ca.crt -CAkey C:\Users\mike\.minikube\ca.key -CAcreateserial  -out deploy_edit.crt -days 365
```
Create user in kubernetes
```bash
kubectl config set-credentials deploy_view --client-certificate=deploy_view.crt --client-key=deploy_view.key
kubectl config set-credentials deploy_edit --client-certificate=deploy_edit.crt --client-key=deploy_edit.key
```
Set context for user
```bash
kubectl config set-context deploy_view --cluster=minikube --user=deploy_view
kubectl config set-context deploy_edit --cluster=minikube --user=deploy_edit
```
Check new rows in config
```bash
Change path
- name: deploy_view
  user:
    client-certificate: C:\Users\mike\Training\Kubernetes\devops-school-k8s\04\deploy_view.crt
    client-key: C:\Users\mike\Training\Kubernetes\devops-school-k8s\04\deploy_view.key
contexts:
- context:
    cluster: minikube
    user: deploy_view
  name: deploy_view

Change path
- name: deploy_edit
  user:
    client-certificate: C:\Users\mike\Training\Kubernetes\devops-school-k8s\04\deploy_edit.crt
    client-key: C:\Users\mike\Training\Kubernetes\devops-school-k8s\04\deploy_edit.key
contexts:
- context:
    cluster: minikube
    user: deploy_edit
  name: deploy_edit
```

Apppy RBAC Role and RoleBinding for deploy_edit and deploy_view users
```bash
kubectl apply -f rbac_rolebinding.yaml
kubectl apply -f rbac_role.yaml
```

Check privileges
```bash
kubectl --context=deploy_view apply -f deploy.yaml
Error from server (Forbidden): error when creating "deploy.yaml": deployments.apps is forbidden: User "deploy_view" cannot create resource "deployments" in API group "apps" in the namespace "default"
```

```bash
kubectl --context=deploy_edit apply -f deploy.yaml
deployment.apps/nginx-deployment created
```

```bash
kubectl --context=deploy_view get po
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-7f84dfb88-sxgr3   1/1     Running   0          21s
nginx-deployment-7f84dfb88-mgs5t   1/1     Running   0          23s
nginx-deployment-7f84dfb88-xsx2h   1/1     Running   0          23s
```

```bash
kubectl --context=deploy_view get deploy
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           90s
```

```bash
kubectl --context=deploy_view get replicaset
NAME                         DESIRED   CURRENT   READY   AGE
nginx-deployment-7f84dfb88   3         3         3       131s
```

```bash
kubectl --context=deploy_edit get svc
Error from server (Forbidden): services is forbidden: User "deploy_edit" cannot list resource "services" in API group "" in the namespace "default"
```

```bash
kubectl --context=deploy_view get svc
Error from server (Forbidden): services is forbidden: User "deploy_view" cannot list resource "services" in API group "" in the namespace "default"
```

```bash
kubectl --context=deploy_view delete po nginx-deployment-7f84dfb88-sxgr3
Error from server (Forbidden): pods "nginx-deployment-7f84dfb88-sxgr3" is forbidden: User "deploy_view" cannot delete resource "pods" in API group "" in the namespace "default"
```

```bash
kubectl --context=deploy_edit delete po nginx-deployment-7f84dfb88-sxgr3
pod "nginx-deployment-7f84dfb88-sxgr3" deleted
```

```bash
kubectl --context=deploy_view delete deploy nginx-deployment
Error from server (Forbidden): deployments.apps "nginx-deployment" is forbidden: User "deploy_view" cannot delete resource "deployments" in API group "apps" in the namespace "default"
```

```bash
kubectl --context=deploy_edit delete deploy nginx-deployment
deployment.apps "nginx-deployment" deleted
```

## Create namespace prod. Create users prod_admin, prod_view. Give the user prod_admin admin rights on ns prod, give the user prod_view only view rights on namespace prod.

Create private key for users
```bash
openssl genrsa -out prod_admin.key 2048
openssl genrsa -out prod_view.key 2048
```
Create a certificate signing request
```bash
openssl req -new -key prod_admin.key -out prod_admin.csr -subj "/CN=prod_admin"
openssl req -new -key prod_view.key -out prod_view.csr -subj "/CN=prod_view"
```
Sign the CSR in the Kubernetes CA (in our case in .minikube).
```bash
openssl x509 -req -in prod_admin.csr -CA C:\Users\mike\.minikube\ca.crt -CAkey C:\Users\mike\.minikube\ca.key -CAcreateserial  -out prod_admin.crt -days 365
openssl x509 -req -in prod_view.csr -CA C:\Users\mike\.minikube\ca.crt -CAkey C:\Users\mike\.minikube\ca.key -CAcreateserial  -out prod_view.crt -days 365
```
Create user in kubernetes
```bash
kubectl config set-credentials prod_admin --client-certificate=prod_admin.crt --client-key=prod_admin.key
kubectl config set-credentials prod_view --client-certificate=prod_view.crt --client-key=prod_view.key
```
Set context for user
```bash
kubectl config set-context prod_admin --cluster=minikube --user=prod_admin
kubectl config set-context prod_view --cluster=minikube --user=prod_view
```
Create prode namespace
```bash
kubectl create ns prod
```
Apply role for users
```bash
kubectl apply -f prod_role_binding.yaml
```
Check privileges
```bash
kubectl --context=prod_view get all
Error from server (Forbidden): pods is forbidden: User "prod_view" cannot list resource "pods" in API group "" in the namespace "default"
Error from server (Forbidden): replicationcontrollers is forbidden: User "prod_view" cannot list resource "replicationcontrollers" in API group "" in the namespace "default"
Error from server (Forbidden): services is forbidden: User "prod_view" cannot list resource "services" in API group "" in the namespace "default"
Error from server (Forbidden): daemonsets.apps is forbidden: User "prod_view" cannot list resource "daemonsets" in API group "apps" in the namespace "default"
Error from server (Forbidden): deployments.apps is forbidden: User "prod_view" cannot list resource "deployments" in API group "apps" in the namespace "default"
Error from server (Forbidden): replicasets.apps is forbidden: User "prod_view" cannot list resource "replicasets" in API group "apps" in the namespace "default"
Error from server (Forbidden): statefulsets.apps is forbidden: User "prod_view" cannot list resource "statefulsets" in API group "apps" in the namespace "default"
Error from server (Forbidden): horizontalpodautoscalers.autoscaling is forbidden: User "prod_view" cannot list resource "horizontalpodautoscalers" in API group "autoscaling" in the namespace "default"
Error from server (Forbidden): cronjobs.batch is forbidden: User "prod_view" cannot list resource "cronjobs" in API group "batch" in the namespace "default"
Error from server (Forbidden): jobs.batch is forbidden: User "prod_view" cannot list resource "jobs" in API group "batch" in the namespace "default"
```

```bash
kubectl --context=prod_view get all -n prod
No resources found in prod namespace.
```

```bash
kubectl --context=prod_view -n prod apply -f deploy.yaml
Error from server (Forbidden): error when creating "deploy.yaml": deployments.apps is forbidden: User "prod_view" cannot create resource "deployments" in API group "apps" in the namespace "prod"
```

```bash
kubectl --context=prod_admin -n prod apply -f deploy.yaml
deployment.apps/nginx-deployment created
```

```bash
kubectl --context=prod_view get all -n prod
NAME                                   READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7f84dfb88-jtlv7   1/1     Running   0          51s
pod/nginx-deployment-7f84dfb88-76kxt   1/1     Running   0          51s
pod/nginx-deployment-7f84dfb88-ujlem   1/1     Running   0          51s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           51s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7f84dfb88   3         3         3       51s
```

```bash
kubectl --context=prod_view -n prod delete pod/nginx-deployment-7f84dfb88-jtlv7
Error from server (Forbidden): pods "nginx-deployment-7f84dfb88-jtlv7" is forbidden: User "prod_view" cannot delete resource "pods" in API group "" in the namespace "prod"
```

```bash
kubectl --context=prod_admin -n prod delete pod/nginx-deployment-7f84dfb88-jtlv7
pod "nginx-deployment-7f84dfb88-jtlv7" deleted
```

```bash
kubectl --context=prod_view -n prod delete deployment.apps/nginx-deployment
Error from server (Forbidden): deployments.apps "nginx-deployment" is forbidden: User "prod_view" cannot delete resource "deployments" in API group "apps" in the namespace "prod"
```

```bash
kubectl --context=prod_admin -n prod delete deployment.apps/nginx-deployment
deployment.apps "nginx-deployment" deleted
```

## Create a serviceAccount sa-namespace-admin. Grant full rights to namespace default. Create context, authorize using the created sa, check accesses.

Create sa and set role
```bash
kubectl apply -f sa.yaml
```
Create context at kubeconfig
```bash
export TOKENNAME=$(kubectl -n default get serviceaccount/sa-namespace-admin -o jsonpath='{.secrets[0].name}')
export TOKEN=$(kubectl -n default get secret $TOKENNAME -o jsonpath='{.data.token}' | base64 --decode)
kubectl config set-credentials sa-namespace-admin --token=$TOKEN
```

Set context to SA
```bash
kubectl config set-context --current --user=sa-namespace-admin
```

Check privileges
```bash
kubectl get po -n prod
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:sa-namespace-admin" cannot list resource "pods" in API group "" in the namespace "prod"
```

```bash
kubectl apply -f deploy -n prod
```

```bash
kubectl apply -f deploy.yaml -n prod
Error from server (Forbidden): error when retrieving current configuration of:
Resource: "apps/v1, Resource=deployments", GroupVersionKind: "apps/v1, Kind=Deployment"
Name: "nginx-deployment", Namespace: "prod"
from server for: "deploy.yaml": deployments.apps "nginx-deployment" is forbidden: User "system:serviceaccount:default:sa-namespace-admin" cannot get resource "deployments" in API group "apps" in the namespace "prod"
```

```bash
kubectl apply -f deploy.yaml
deployment.apps/nginx-deployment created
```

```bash
kubectl get all
NAME                                   READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7f84dfb88-76jmv   1/1     Running   0          12s
pod/nginx-deployment-7f84dfb88-mjtvw   1/1     Running   0          12s
pod/nginx-deployment-7f84dfb88-uy7sg   1/1     Running   0          12s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   107m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           12s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7f84dfb88   3         3         3       12s
```

```bash
kubectl delete pod/nginx-deployment-7f84dfb88-76jmv
pod "nginx-deployment-7f84dfb88-76jmv" deleted
```

```bash
kubectl delete deploy nginx-deployment
deployment.apps "nginx-deployment" deleted
```