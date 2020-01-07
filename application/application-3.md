## Know how to scale applications


- use `kubectl scale`

```
ubuntu@k8smaster:~/application$ kubectl get deployment
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           8h
ubuntu@k8smaster:~/application$ kubectl scale deployment nginx --replicas=5
deployment.apps/nginx scaled
ubuntu@k8smaster:~/application$ kubectl get deployment
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   5/5     5            5           8h
```

- use `kubectl autoscale`

```
ubuntu@k8smaster:~/application$ kubectl autoscale deployment nginx --min=1 --max=7
horizontalpodautoscaler.autoscaling/nginx autoscaled
ubuntu@k8smaster:~/application$ kubectl get hpa
NAME    REFERENCE          TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
nginx   Deployment/nginx   <unknown>/80%   1         7         5          3m53s

```

- refer
[scaling-your-application](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#scaling-your-application)
[pod autoscale](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale)
