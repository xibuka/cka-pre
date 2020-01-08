## Facilitate operating system upgrades 

- evict pods from one node. 
When we need to take a node down for maintenance, Kubernetes makes it easy to 
evict the pods on that node, take it down, and then continue scheduling pods 
after the maintenance is complete.

```
ubuntu@k8smaster:~$ kubectl get pod -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
nginx-555786ff8-6lv8p   1/1     Running   0          11s   10.244.2.33   k8snode2    <none>           <none>
nginx-555786ff8-bk257   1/1     Running   0          11s   10.244.1.76   k8snode1    <none>           <none>
nginx-555786ff8-c4kr8   1/1     Running   0          11s   10.244.1.77   k8snode1    <none>           <none>
nginx-555786ff8-jw5f9   1/1     Running   0          11s   10.244.0.50   k8smaster   <none>           <none>
nginx-555786ff8-smcjd   1/1     Running   0          11s   10.244.0.51   k8smaster   <none>           <none>
ubuntu@k8smaster:~$ 
ubuntu@k8smaster:~$ kubectl drain k8snode1 
node/k8snode1 cordoned
evicting pod "nginx-555786ff8-bk257"
evicting pod "nginx-555786ff8-c4kr8"
evicting pod "coredns-6955765f44-k6czw"
evicting pod "test-659588d944-xzmg2"

pod/coredns-6955765f44-k6czw evicted
pod/nginx-555786ff8-c4kr8 evicted
pod/nginx-555786ff8-bk257 evicted
pod/test-659588d944-xzmg2 evicted
node/k8snode1 evicted
ubuntu@k8smaster:~$ 
ubuntu@k8smaster:~$ kubectl get node
NAME        STATUS                     ROLES    AGE   VERSION
k8smaster   Ready                      master   13d   v1.17.0
k8snode1    Ready,SchedulingDisabled   <none>   12d   v1.17.0
k8snode2    Ready                      <none>   40h   v1.17.0
ubuntu@k8smaster:~$ kubectl get pod -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
nginx-555786ff8-5b86v   1/1     Running   0          20m   10.244.2.34   k8snode2    <none>           <none>
nginx-555786ff8-6lv8p   1/1     Running   0          22m   10.244.2.33   k8snode2    <none>           <none>
nginx-555786ff8-9gv5d   1/1     Running   0          20m   10.244.0.52   k8smaster   <none>           <none>
nginx-555786ff8-jw5f9   1/1     Running   0          22m   10.244.0.50   k8smaster   <none>           <none>
nginx-555786ff8-smcjd   1/1     Running   0          22m   10.244.0.51   k8smaster   <none>           <none>
```

Now we can do maintenance for k8snode1

- remove the node from cluster

```
ubuntu@k8smaster:~$ kubectl get node
NAME        STATUS   ROLES    AGE   VERSION
k8smaster   Ready    master   13d   v1.17.0
k8snode1    Ready    <none>   12d   v1.17.0
k8snode2    Ready    <none>   40h   v1.17.0
ubuntu@k8smaster:~$ kubectl delete node k8snode1
node "k8snode1" deleted
ubuntu@k8smaster:~$ 
ubuntu@k8smaster:~$ kubectl get node
NAME        STATUS   ROLES    AGE   VERSION
k8smaster   Ready    master   13d   v1.17.0
k8snode2    Ready    <none>   40h   v1.17.0
```

- Add a node to cluster

In master node
```
ubuntu@k8smaster:~$ sudo kubeadm token create --print-join-command
W0108 07:12:58.362049   17845 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0108 07:12:58.362081   17845 validation.go:28] Cannot validate kubelet config - no validator is available
kubeadm join 20.0.1.54:6443 --token 8nlfu7.mi6hap0pfr4h1aas     --discovery-token-ca-cert-hash sha256:44aac704627d7d28d529bb2b0fa7dea982b02ff0e1bb9b26941c4fa56bf3144e
```

In the new node
```
ubuntu@k8snode1:~$ sudo kubeadm reset cleanup-node # if needed
ubuntu@k8snode1:~$ sudo kubeadm join 20.0.1.54:6443 --token 8nlfu7.mi6hap0pfr4h1aas     --discovery-token-ca-cert-hash sha256:44aac704627d7d28d529bb2b0fa7dea982b02ff0e1bb9b26941c4fa56bf3144e 
```

Then we should see new node is joined 
```
ubuntu@k8smaster:~$ kubectl get node
NAME        STATUS   ROLES    AGE   VERSION
k8smaster   Ready    master   13d   v1.17.0
k8snode1    Ready    <none>   68s   v1.17.0
k8snode2    Ready    <none>   40h   v1.17.0
```

- refer

[maintenance a node](https://kubernetes.io/docs/tasks/administer-cluster/cluster-management/#maintenance-on-a-node)

[kubeadm reset](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-reset/_
