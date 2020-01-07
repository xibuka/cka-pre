## Understand how to monitor all cluster components

- Metrics Server
We can monitor the CPU and memory utilization of our pods and nodes by metrics
server.
In the CKA exam, it should be installed already so you don't need to install it.

```
ubuntu@k8smaster:~$ git clone https://github.com/linuxacademy/metrics-server
Cloning into 'metrics-server'...
remote: Enumerating objects: 9589, done.
remote: Total 9589 (delta 0), reused 0 (delta 0), pack-reused 9589
Receiving objects: 100% (9589/9589), 11.23 MiB | 5.12 MiB/s, done.
Resolving deltas: 100% (4853/4853), done.
ubuntu@k8smaster:~$ kubectl apply -f metrics-server/deploy/1.8+/
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
deployment.apps/metrics-server created
ubuntu@k8smaster:~$ kubectl get --raw /apis/metrics.k8s.io/
{"kind":"APIGroup","apiVersion":"v1","name":"metrics.k8s.io","versions":[{"groupVersion":"metrics.k8s.io/v1beta1","version":"v1beta1"}],"preferredVersion":{"groupVersion":"metrics.k8s.io/v1beta1","version":"v1beta1"}}
```

After install, we can use `kubectl top` command to monitor the resources.

```
ubuntu@k8smaster:~$ kubectl top node
NAME        CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
k8smaster   82m          4%     891Mi           23%       
k8snode1    38m          1%     478Mi           12%       
k8snode2    34m          1%     476Mi           12%
ubuntu@k8smaster:~$ kubectl top pods
NAME                    CPU(cores)   MEMORY(bytes)
busybox                 0m           0Mi
nginx-555786ff8-2rj46   0m           3Mi
nginx-555786ff8-sds7t   0m           2Mi
nginx-555786ff8-srfdc   0m           3Mi
nginx-555786ff8-tq9sl   0m           2Mi
nginx-555786ff8-wg8ft   0m           3Mi
```

other options

```
ubuntu@k8smaster:~$ kubectl top pods -n kube-system
NAME                                CPU(cores)   MEMORY(bytes)   
coredns-6955765f44-k6czw            2m           5Mi             
coredns-6955765f44-psbp8            2m           5Mi             
etcd-k8smaster                      12m          37Mi            
kube-apiserver-k8smaster            20m          241Mi           
kube-controller-manager-k8smaster   8m           34Mi            
kube-flannel-ds-amd64-69b9t         2m           8Mi             
kube-flannel-ds-amd64-bfg88         2m           8Mi             
kube-flannel-ds-amd64-mz44q         2m           9Mi             
kube-proxy-8kv2z                    1m           11Mi            
kube-proxy-rmb6n                    1m           12Mi            
kube-proxy-w4xsz                    1m           12Mi            
kube-scheduler-k8smaster            3m           10Mi            
metrics-server-f76f648c7-4wnsz      1m           13Mi 

ubuntu@k8smaster:~$ kubectl top pods --all-namespaces
NAMESPACE     NAME                                CPU(cores)   MEMORY(bytes)
default       busybox                             0m           0Mi
default       nginx-555786ff8-2rj46               0m           3Mi
default       nginx-555786ff8-sds7t               0m           2Mi
default       nginx-555786ff8-srfdc               0m           3Mi
default       nginx-555786ff8-tq9sl               0m           2Mi
default       nginx-555786ff8-wg8ft               0m           3Mi
kube-system   coredns-6955765f44-k6czw            2m           5Mi
kube-system   coredns-6955765f44-psbp8            2m           5Mi
kube-system   etcd-k8smaster                      12m          37Mi
kube-system   kube-apiserver-k8smaster            20m          241Mi
kube-system   kube-controller-manager-k8smaster   8m           34Mi
kube-system   kube-flannel-ds-amd64-69b9t         2m           8Mi
kube-system   kube-flannel-ds-amd64-bfg88         2m           8Mi
kube-system   kube-flannel-ds-amd64-mz44q         2m           9Mi
kube-system   kube-proxy-8kv2z                    1m           11Mi
kube-system   kube-proxy-rmb6n                    1m           12Mi
kube-system   kube-proxy-w4xsz                    1m           12Mi
kube-system   kube-scheduler-k8smaster            3m           10Mi
kube-system   metrics-server-f76f648c7-4wnsz      1m           13Mi
my-ns         test-659588d944-xzmg2               1m           9Mi

ubuntu@k8smaster:~$ kubectl top pods --all-namespaces -l run=nginx
NAMESPACE   NAME                    CPU(cores)   MEMORY(bytes)
default     nginx-555786ff8-2rj46   0m           3Mi
default     nginx-555786ff8-sds7t   0m           2Mi
default     nginx-555786ff8-srfdc   0m           3Mi
default     nginx-555786ff8-tq9sl   0m           2Mi
default     nginx-555786ff8-wg8ft   0m           3Mi

ubuntu@k8smaster:~$ kubectl top pod busybox
NAME      CPU(cores)   MEMORY(bytes)
busybox   0m           0Mi

ubuntu@k8smaster:~$ kubectl top pods busybox --containers
POD       NAME      CPU(cores)   MEMORY(bytes)
busybox   busybox   0m           0Mi
```
- refer
[Monitor Node Health](https://kubernetes.io/docs/tasks/debug-application-cluster/monitor-node-health/)
[Resource Usage Monitoring](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/)
[prometheus-operator](https://github.com/coreos/prometheus-operator)
[monitoring Kubernetes(part1)](https://sysdig.com/blog/monitoring-kubernetes-with-sysdig-cloud/)
[Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)
[Troubleshoot Clusters](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)
