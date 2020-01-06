## Manually schedule a pod without a scheduler

- static pod
A static pod can be created before scheduler is running.
This is done by `kubelet`. There is a parameter `--pod-manifest-path` which is 
used to specify a directy for `kubelet` to monitor.
When a pod's yaml file exist in this directory, `kubelet` will create it.
Usually it's useful when init the cluster

- create a static pod
put below file under the monitor directory(/etc/kubelet.d/ in this example)
```
cat <<EOF >/etc/kubelet.d/static-web.yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-web
  labels:
    role: myrole
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
EOF
```

we can see the pod after a while
```
# kubectl get pods
NAME                                            READY     STATUS    RESTARTS   AGE
static-web-cn-shenzhen.i-wz9jbz8fkh7uiqixg4ct   1/1       Running   0          14m

```
static pod is not controled by api server, so it can not be deleted by kubectl.
To delete a static pod we can just delete the yaml file itself.

```
[root@iZwz9jb]# mv static-web.yaml /tmp/
[root@iZwz9jb]# kubectl get pods
No resources found.

```

- refer
[static pod](https://kubernetes.io/docs/tasks/administer-cluster/static-pod/)
