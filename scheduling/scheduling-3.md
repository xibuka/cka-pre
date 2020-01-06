## Understand how resource limits can affect Pod scheduling

Limits can limit the resource used by the container.
Requests can tell the scheduler how many resources the container need.

- view the capacity and the allocatable info from a node
```
ubuntu@k8smaster:~$ kubectl describe node k8snode2
Name:               k8snode2
...
Capacity:
  cpu:                2
  ephemeral-storage:  19092136Ki
  hugepages-2Mi:      0
  memory:             4038468Ki
  pods:               110
Allocatable:
  cpu:                2
  ephemeral-storage:  17595312509
  hugepages-2Mi:      0
  memory:             3936068Ki
  pods:               110
```

- a pod yaml for a pod with requests

```
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod1
spec:
  nodeSelector:
    kubernetes.io/hostname: "chadcrowell3c.mylabserver.com"
  containers:
  - image: busybox
    command: ["dd", "if=/dev/zero", "of=/dev/null"]
    name: pod1
    resources:
      requests:
        cpu: 800m
        memory: 20Mi
```

- a pod yaml for a pod with limits

requests is automatically set with the same value of limits if no requests set

```
apiVersion: v1
kind: Pod
metadata:
  name: limited-pod
spec:
  containers:
  - image: busybox
    command: ["dd", "if=/dev/zero", "of=/dev/null"]
    name: main
    resources:
      limits:
        cpu: 1
        memory: 20Mi
```

- create the LimitRange object to set the default limits

```
admin/resource/cpu-defaults-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: default-cpu-demo
spec:
  containers:
  - name: default-cpu-demo-ctr
    image: nginx
```

一个简单的针对pod限制内存和cpu的例子模板如下:

```
apiVersion: v1
kind: Pod
metadata:
  name: cpu-demo-2
  namespace: cpu-example
spec:
  containers:
  - name: cpu-demo-ctr-2
    image: vish/stress
    resources:
      limits:
        cpu: "1"
      requests:
        cpu: "0.5"
    args:
    - -cpus
    - "2"
```

## reference
[Configure Default CPU Requests and Limits](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/)
[assign-cpu-resource/](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)
[memory-default-namespace/](https://kubernetes.io/docs/tasks/administer-cluster/memory-default-namespace/)
[Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
[Configure Quotas for API Objects](https://kubernetes.io/docs/tasks/administer-cluster/quota-api-object/)
