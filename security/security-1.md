## Know how to configure authentication and authorization

- Role and Rolebinding


```
ubuntu@k8smaster:~$ kubectl create ns web
namespace/web created

ubuntu@k8smaster:~/auth$ kubectl create role service-reader --verb=get,list --resource=services -n web
role.rbac.authorization.k8s.io/service-reader created
ubuntu@k8smaster:~/auth$ 
ubuntu@k8smaster:~/auth$ kubectl create rolebinding test --role=service-reader --serviceaccount=web:default -n web
rolebinding.rbac.authorization.k8s.io/test created

ubuntu@k8smaster:~/auth$ curl localhost:8001/api/v1/namespaces/web/services
{
  "kind": "ServiceList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/namespaces/web/services",
    "resourceVersion": "229377"
  },
  "items": []
}
```

- ClusterRole and ClusterRoleBinding

```

```
ubuntu@k8smaster:~/auth$ kubectl create clusterrole pv-reader --verb=get,list --resource=persistentvolumes -n web
clusterrole.rbac.authorization.k8s.io/pv-reader created
ubuntu@k8smaster:~/auth$ kubectl create clusterrolebinding test --clusterrole=pv-reader --serviceaccount=web:default
clusterrolebinding.rbac.authorization.k8s.io/test created

ubuntu@k8smaster:~/auth$ curl localhost:8001/api/v1/persistentvolumes
{
  "kind": "PersistentVolumeList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/persistentvolumes",
    "resourceVersion": "230059"
  },
  "items": []
}ubuntu@k8smaster:~/auth$
```

- refer

[Authenticating](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)

[RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

