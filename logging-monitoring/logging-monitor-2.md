## Understand how to monitor applications

- monitor resource of the application
```
ubuntu@k8smaster:~$ kubectl top pods --all-namespaces -l run=nginx
NAMESPACE   NAME                    CPU(cores)   MEMORY(bytes)   
default     nginx-555786ff8-2rj46   0m           3Mi             
default     nginx-555786ff8-sds7t   0m           2Mi             
default     nginx-555786ff8-srfdc   0m           3Mi             
default     nginx-555786ff8-tq9sl   0m           2Mi             
default     nginx-555786ff8-wg8ft   0m           3Mi             
```

- liveness probe

```
ubuntu@k8smaster:~$ cat liveness.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: liveness
spec:
  containers:
  - image: linuxacademycontent/kubeserve
    name: kubeserve
    livenessProbe:
      httpGet:
        path: /
        port: 80
ubuntu@k8smaster:~$ kubectl create -f liveness.yaml 
pod/liveness created
ubuntu@k8smaster:~$ kubectl get pod
NAME                    READY   STATUS    RESTARTS   AGE
liveness                1/1     Running   0          61s
```

- readness probe

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-readiness
spec:
  replicas: 5
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        readinessProbe:
          exec:
            command:
              - ls
              - /var/ready
ubuntu@k8smaster:~$ kubectl create -f readiness.yaml 
deployment.apps/nginx-readiness created
ubuntu@k8smaster:~$ kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
busybox                           1/1     Running   3          12h
liveness                          1/1     Running   0          89m
nginx-readiness-6bcfbd848-4h7ft   0/1     Running   0          84s
nginx-readiness-6bcfbd848-5txgg   0/1     Running   0          85s
nginx-readiness-6bcfbd848-8vcr5   0/1     Running   0          84s
nginx-readiness-6bcfbd848-h6vxt   0/1     Running   0          84s
nginx-readiness-6bcfbd848-xhcnx   0/1     Running   0          84s
```

`nginx-readiness` pods are not READY because the readiness probe is not
success.
It will become ready when we touch the `/var/ready` file, which was defined in
the yaml file.

```
ubuntu@k8smaster:~$ kubectl exec nginx-readiness-6bcfbd848-4h7ft -- touch /var/ready
ubuntu@k8smaster:~$
ubuntu@k8smaster:~$ kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
busybox                           1/1     Running   3          12h
liveness                          1/1     Running   0          90m
nginx-readiness-6bcfbd848-4h7ft   1/1     Running   0          2m31s
nginx-readiness-6bcfbd848-5txgg   0/1     Running   0          2m32s
nginx-readiness-6bcfbd848-8vcr5   0/1     Running   0          2m31s
nginx-readiness-6bcfbd848-h6vxt   0/1     Running   0          2m31s
nginx-readiness-6bcfbd848-xhcnx   0/1     Running   0          2m31s

```

- refer 
[Container probes](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
[Configure probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
