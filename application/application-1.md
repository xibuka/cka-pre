## Understand Deployment and how to perform rolling updates and rollbacks

- create a deployment with record(for rollbacks)

```
ubuntu@k8smaster:~/application$ cat nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.7.9
          ports:
            - containerPort: 80
ubuntu@k8smaster:~/application$ kubectl create -f nginx.yaml --record
deployment.apps/nginx created
```

A replicaSet will be created to create pod
```
ubuntu@k8smaster:~/application$ kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7cccdd79bf-658gg   1/1     Running   0          104s
nginx-7cccdd79bf-lrkpr   1/1     Running   0          104s
nginx-7cccdd79bf-lvfv9   1/1     Running   0          104s
ubuntu@k8smaster:~/application$ kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
nginx-7cccdd79bf   3         3         3       2m20s
```

- scale the deployment

```
ubuntu@k8smaster:~/application$ kubectl scale deployment nginx --replicas=5
deployment.apps/nginx scaled
ubuntu@k8smaster:~/application$ 
ubuntu@k8smaster:~/application$ kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7cccdd79bf-658gg   1/1     Running   0          4m32s
nginx-7cccdd79bf-lrkpr   1/1     Running   0          4m32s
nginx-7cccdd79bf-lvfv9   1/1     Running   0          4m32s
nginx-7cccdd79bf-s67qm   1/1     Running   0          5s
nginx-7cccdd79bf-tbqt8   1/1     Running   0          5s
```

Or we can edit the deployment object itself to change the `spec.replicas`

```
ubuntu@k8smaster:~/application$ kubectl edit deployment nginx 
deployment.apps/nginx edited
```

- update the container image version

update the yaml file and run `kubectl apply -f` or `kubectl replace -f`
```
ubuntu@k8smaster:~/application$ vim nginx.yaml
ubuntu@k8smaster:~/application$
ubuntu@k8smaster:~/application$ kubectl apply -f nginx.yaml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.apps/nginx configured
```

update the image version by `kubectl set image`
```
ubuntu@k8smaster:~/application$ kubectl set image deployment nginx nginx=nginx:1.9.1 --record
deployment.apps/nginx image updated

```

- check the revision of the deployment

```
ubuntu@k8smaster:~/application$ kubectl rollout history deployment nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
2         kubectl create --filename=nginx.yaml --record=true
4         kubectl set image deployment nginx nginx=nginx:1.9.1 --record=true
5         kubectl set image deployment nginx nginx=nginx:1.7.9 --record=true
```

- undo a update 
this will undo the update and back to the previous version

```
ubuntu@k8smaster:~/application$ kubectl rollout undo deployment nginx
deployment.apps/nginx rolled back
ubuntu@k8smaster:~/application$
ubuntu@k8smaster:~/application$ kubectl get pod
NAME                     READY   STATUS              RESTARTS   AGE
nginx-754b687b4c-cpnr6   0/1     ContainerCreating   0          2s
nginx-754b687b4c-qmhjx   1/1     Running             0          4s
nginx-7cccdd79bf-gbw85   1/1     Running             0          115s
nginx-7cccdd79bf-mb59j   0/1     Terminating         0          117s
nginx-7cccdd79bf-pdrxw   1/1     Running             0          113s
```

Or roll back to a specify version
```
ubuntu@k8smaster:~/application$ kubectl rollout undo deployment nginx --to-revision=2
deployment.apps/nginx rolled back
ubuntu@k8smaster:~/application$
ubuntu@k8smaster:~/application$ kubectl get pod
NAME                     READY   STATUS              RESTARTS   AGE
nginx-754b687b4c-6jl7v   1/1     Running             0          69s
nginx-754b687b4c-cpnr6   1/1     Running             0          71s
nginx-754b687b4c-qmhjx   1/1     Terminating         0          73s
nginx-d9dc584f-psjkp     1/1     Running             0          3s
nginx-d9dc584f-tdmfs     0/1     ContainerCreating   0          1s

```

- pause and resume an rollout update

```
ubuntu@k8smaster:~/application$ kubectl rollout pause deployment nginx
deployment.apps/nginx paused

ubuntu@k8smaster:~/application$ kubectl rollout resume deployment nginx
deployment.apps/nginx resumed

- refer
[Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)
