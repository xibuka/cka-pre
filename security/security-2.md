## Understand Kubernets security primitives


- service account

```
ubuntu@k8smaster:~$ kubectl config set-credentials wshi --username=wshi --password=password
User "wshi" set.
ubuntu@k8smaster:~$ kubectl create clusterrolebinding cluster-system-anonymous --clusterrole=cluster-admin --user=system:anonymous
clusterrolebinding.rbac.authorization.k8s.io/cluster-system-anonymous created
ubuntu@k8smaster:~$
ubuntu@k8smaster:~$ cd /etc/kubernetes/pki/
ubuntu@k8smaster:/etc/kubernetes/pki$ ls
apiserver-etcd-client.crt  apiserver-kubelet-client.crt  apiserver.crt  ca.crt  etcd                front-proxy-ca.key      front-proxy-client.key  sa.pub
apiserver-etcd-client.key  apiserver-kubelet-client.key  apiserver.key  ca.key  front-proxy-ca.crt  front-proxy-client.crt  sa.key
ubuntu@k8smaster:/etc/kubernetes/pki$ logout
Connection to 20.0.1.54 closed.
wshi@ttbox:~$ scp ubuntu@20.0.1.54:/etc/kubernetes/pki/ca.crt ./
ca.crt
```

Use above `ca.crt` file in another machine
```
wshi@ttbox:~$ kubectl config set-cluster kubernetes --server=https://20.0.1.54:6443 --certificate-authority=ca.crt --embed-certs=true
Cluster "kubernetes" set.
wshi@ttbox:~$ 
wshi@ttbox:~$ kubectl config set-credentials wshi --username=wshi --password=password
User "wshi" set.
wshi@ttbox:~$ kubectl config set-context kubernetes --cluster=kubernetes --user=wshi --namespace=default
Context "kubernetes" created.
wshi@ttbox:~$ 
wshi@ttbox:~$ kubectl config use-context kubernetes
Switched to context "kubernetes".
wshi@ttbox:~$ 
wshi@ttbox:~$ kubectl get node
NAME        STATUS   ROLES    AGE     VERSION
k8smaster   Ready    master   14d     v1.17.0
k8snode1    Ready    <none>   25h     v1.17.0
k8snode2    Ready    <none>   2d18h   v1.17.0
```

%TODO
Some details about `set-credentials` and `kubectl config`

对一个集群增加安全性，主要从控制所有针对API的操作和ETCD数据来进行，官方文档进行了很详细的介绍，

[Securing a Cluster](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/)。

另外，对于Pod的安全性，1.9版本也加入了新的特性
[Pod Security Policies](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)
