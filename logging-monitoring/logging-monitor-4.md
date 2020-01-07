## Manage application logs

- get the logs from pod

```
kubectl logs
kubectl logs podname
kubectl logs podname -c container-name
kubectl logs podname --all-containers=true
```

- get a streaming log
```
kubectl logs -f podname -c countainer-name
```

- get a part of the log
tail the log

```
kubectl logs --tail=20 podname
```

get the log since 1 hour ago
```
kubectl logs --since=1h podname
```

- use ELK
read https://kubernetes.io/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana/

- refer
[logs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs)
