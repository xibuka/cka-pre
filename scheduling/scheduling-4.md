## Understand how to run multiple schedulers and how to configure Pods to use them

- create a new scheduler
see https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/

- Specify schedulers for pods
Use `schedulerName` under `spec` to specify which scheduler to use.

```
apiVersion: v1
kind: Pod
metadata:
  name: annotation-second-scheduler
  labels:
    name: multischeduler-example
spec:
  schedulerName: my-scheduler
  containers:
  - name: pod-with-second-annotation-container
    image: k8s.gcr.io/pause:2.0
```

- refer 
[multiple schedulers](https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/)
