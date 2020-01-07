## Understand the primitivies necessary to create a self-healing application
自动恢复的pod，目前Deployment 或者ReplicaSet等，你只要指定了具体的`replica`数目，
控制器就会默认帮你维持这个pod的数量，如果有一个pod 因为某种原因挂掉，
控制器就会在其他地方帮你自动的在创建出来一个新的pod，一直保持pod数量的稳定性。这就是控制器的目的所在。

- ReplicaSets
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

- StatefulSets

```
# a Headless service is used to control the network domain
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

- refer 

[ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

[StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
