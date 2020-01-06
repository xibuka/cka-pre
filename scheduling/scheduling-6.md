## Display scheduler events

- `kubectl describe <resource>`

```
ubuntu@k8smaster:~$ kubectl describe pod nginx-555786ff8-wg8ft
Name:         nginx-555786ff8-wg8ft
Namespace:    default
Priority:     0
Node:         k8smaster/20.0.1.54
Start Time:   Mon, 06 Jan 2020 16:00:22 +0000
Labels:       pod-template-hash=555786ff8
              run=nginx
Annotations:  <none>
Status:       Running
IP:           10.244.0.36
IPs:
  IP:           10.244.0.36
Controlled By:  ReplicaSet/nginx-555786ff8
Containers:
  nginx:
    Container ID:   docker://675b5fbe15ef48f063b22121c726487b1c7a41a3988839040932cc43570647c5
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:b2d89d0a210398b4d1120b3e3a7672c16a4ba09c2c4a0395f18b9f7999b768f2
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 06 Jan 2020 16:00:30 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-g4zrc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-g4zrc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-g4zrc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From                Message
  ----    ------     ----       ----                -------
  Normal  Scheduled  <unknown>  default-scheduler   Successfully assigned default/nginx-555786ff8-wg8ft to k8smaster
  Normal  Pulling    45s        kubelet, k8smaster  Pulling image "nginx"
  Normal  Pulled     38s        kubelet, k8smaster  Successfully pulled image "nginx"
  Normal  Created    38s        kubelet, k8smaster  Created container nginx
  Normal  Started    38s        kubelet, k8smaster  Started container nginx
```
The event is at the button

- `kubectl get events`

```
ubuntu@k8smaster:~$ kubectl get events
LAST SEEN   TYPE     REASON                    OBJECT                       MESSAGE
47m         Normal   Starting                  node/k8smaster               Starting kubelet.
47m         Normal   NodeHasSufficientMemory   node/k8smaster               Node k8smaster status is now: NodeHasSufficientMemory
47m         Normal   NodeHasNoDiskPressure     node/k8smaster               Node k8smaster status is now: NodeHasNoDiskPressure
47m         Normal   NodeHasSufficientPID      node/k8smaster               Node k8smaster status is now: NodeHasSufficientPID
```
We can see some cluster level events by this 

- `kubectl logs <pod name>`

```
ubuntu@k8smaster:~$ kubectl logs nginx-555786ff8-srfdc
10.244.0.0 - - [06/Jan/2020:16:05:14 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.58.0" "-"
10.244.0.0 - - [06/Jan/2020:16:05:21 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.58.0" "-"
```
