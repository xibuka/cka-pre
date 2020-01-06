## Use label selectors to schedule Pods

- Label a node and confirm it

```
ubuntu@k8smaster:~$ kubectl label node k8smaster zone=zone0
node/k8snode1 labeled
ubuntu@k8smaster:~$ kubectl label node k8snode1 zone=zone1
node/k8snode1 labeled
```

2 ways to confirm the label on the node
```
ubuntu@k8smaster:~$ kubectl get node --show-labels
NAME        STATUS   ROLES    AGE   VERSION   LABELS
k8smaster   Ready    master   11d   v1.17.0   <...>,zone=zone0
k8snode1    Ready    <none>   10d   v1.17.0   <...>,zone=zone1
```

```
ubuntu@k8smaster:~$ kubectl get node -l zone=zone0
NAME        STATUS   ROLES    AGE   VERSION
k8smaster   Ready    master   11d   v1.17.0
ubuntu@k8smaster:~$ kubectl get node -l zone=zone1
NAME       STATUS   ROLES    AGE   VERSION
k8snode1   Ready    <none>   10d   v1.17.0
```

- Schedule a pod to the node via `nodeSelector`

```
ubuntu@k8smaster:~$ kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > nginx.pod.yaml
ubuntu@k8smaster:~$ vim nginx.pod.yaml
ubuntu@k8smaster:~$ cat nginx.pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
  nodeSelector:
    zone: zone1
```

- Schedule the pod of a deployment via node affinity
50% on worker node(zone1), 50% on master node(zone0)

```
ubuntu@k8smaster:~$ kubectl run nginx --image=nginx --dry-run -o yaml > nginx.deployment.yaml
ubuntu@k8smaster:~$ vim nginx.deployment.yaml
ubuntu@k8smaster:~$ cat nginx.deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  replicas: 4
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - weight: 80
            preference:
              matchExpressions:
              - key: zone
                operator: In
                values:
                - zone1
          - weight: 20
            preference:
              matchExpressions:
              - key: zone
                operator: In
                values:
                - master
      containers:
      - image: nginx
        name: nginx
ubuntu@k8smaster:~$ kubectl create -f nginx.deployment.yaml 
deployment.apps/nginx created
ubuntu@k8smaster:~$ kubectl get pod -o wide
NAME                    READY   STATUS        RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
nginx-555786ff8-4mqtc   1/1     Running       0          69s   10.244.1.35   k8snode1    <none>           <none>
nginx-555786ff8-bmzc2   1/1     Running       0          69s   10.244.1.34   k8snode1    <none>           <none>
nginx-555786ff8-mc7zk   1/1     Running       0          69s   10.244.0.20   k8smaster   <none>           <none>
nginx-555786ff8-nwmhs   1/1     Running       0          63s   10.244.0.21   k8smaster   <none>           <none>
```

- Refer Links 
[Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/)
[Pod and Node Affinity Rules](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)
[Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
