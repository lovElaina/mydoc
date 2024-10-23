首先通过helm部署k8s-dashboard

```bash
[root@master ~]# kubectl get svc -n kubernetes-dashboard

NAME                  TYPE    CLUSTER-IP    EXTERNAL-IP  PORT(S)             AGE

kubernetes-dashboard-api        ClusterIP  10.100.205.30  <none>    8000/TCP            4d16h

kubernetes-dashboard-auth       ClusterIP  10.98.114.235  <none>    8000/TCP            4d16h

kubernetes-dashboard-kong-manager   NodePort  10.109.40.181  <none>    8002:30136/TCP,8445:31477/TCP  4d16h

kubernetes-dashboard-kong-proxy    ClusterIP  10.101.180.202  <none>    443/TCP             4d16h

kubernetes-dashboard-metrics-scraper  ClusterIP  10.109.241.83  <none>    8000/TCP            4d16h

kubernetes-dashboard-web        NodePort  10.102.60.231  <none>    8000:32580/TCP         4d16h
```

```bash
[root@master ~]# kubectl get pods -n kubernetes-dashboard

NAME                          READY  STATUS  RESTARTS  AGE

kubernetes-dashboard-api-697cf54b86-ttsvh        1/1   Running  0     24h

kubernetes-dashboard-auth-8c596f949-5hlcp        1/1   Running  0     3d15h

kubernetes-dashboard-kong-7696bb8c88-8sw6f       1/1   Running  0     3d15h

kubernetes-dashboard-metrics-scraper-6d659646c4-zhxdz  1/1   Running  0     3d15h

kubernetes-dashboard-web-669b6977f7-kqbzz        1/1   Running  0     24h
```

然后在本地，运行

zyc@bogon / % ssh -L 8443:**10.101.180.202**:443 root@43.128.69.41

注意这里的 **10.101.180.202** 是 kubernetes-dashboard-kong-proxy 的ClusterIP

接下来按照官方指引：

# Creating sample user



In this guide, we will find out how to create a new user using the Service Account mechanism of Kubernetes, grant this user admin permissions and login to Dashboard using a bearer token tied to this user.

For each of the following snippets for `ServiceAccount` and `ClusterRoleBinding`, you should copy them to new manifest files like `dashboard-adminuser.yaml` and use `kubectl apply -f dashboard-adminuser.yaml` to create them.

## Creating a Service Account



We are creating Service Account with the name `admin-user` in namespace `kubernetes-dashboard` first.

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```



## Creating a ClusterRoleBinding



In most cases after provisioning the cluster using `kops`, `kubeadm` or any other popular tool, the `ClusterRole` `cluster-admin` already exists in the cluster. We can use it and create only a `ClusterRoleBinding` for our `ServiceAccount`. If it does not exist then you need to create this role first and grant required privileges manually.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```



## Getting a Bearer Token for ServiceAccount



Now we need to find the token we can use to log in. Execute the following command:

```
kubectl -n kubernetes-dashboard create token admin-user
```



It should print something like:

```
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXY1N253Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwMzAzMjQzYy00MDQwLTRhNTgtOGE0Ny04NDllZTliYTc5YzEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.Z2JrQlitASVwWbc-s6deLRFVk5DWD3P_vjUFXsqVSY10pbjFLG4njoZwh8p3tLxnX_VBsr7_6bwxhWSYChp9hwxznemD5x5HLtjb16kI9Z7yFWLtohzkTwuFbqmQaMoget_nYcQBUC5fDmBHRfFvNKePh_vSSb2h_aYXa8GV5AcfPQpY7r461itme1EXHQJqv-SN-zUnguDguCTjD80pFZ_CmnSE1z9QdMHPB8hoB4V68gtswR1VLa6mSYdgPwCHauuOobojALSaMc3RH7MmFUumAgguhqAkX3Omqd3rJbYOMRuMjhANqd08piDC3aIabINX6gP5-Tuuw2svnV6NYQ
```



Check [Kubernetes docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#manually-create-an-api-token-for-a-serviceaccount) for more information about API tokens for a ServiceAccount.

## Getting a long-lived Bearer Token for ServiceAccount



We can also create a token with the secret which bound the service account and the token will be saved in the Secret:

```
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: "admin-user"   
type: kubernetes.io/service-account-token  
```



After Secret is created, we can execute the following command to get the token which saved in the Secret:

```
kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d
```



Check [Kubernetes docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#manually-create-a-long-lived-api-token-for-a-serviceaccount) for more information about long-lived API tokens for a ServiceAccount.

## Accessing Dashboard



Now copy the token and paste it into the `Enter token` field on the login screen.