## Know to configure network policies

- install a plugin to enable NetworkPolicy

```
ubuntu@k8smaster:~/secruty$ wget -O canal.yaml https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/canal/canal.yaml
...
ubuntu@k8smaster:~/secruty$ kubectl apply -f canal.yaml
...
```

confirm plugin is running

```
ubuntu@k8smaster:~/secruty$ kubectl get all -n kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
pod/canal-784sl                         2/2     Running   0          4m41s
pod/canal-ph2hk                         2/2     Running   0          4m41s
pod/canal-rz6ps                         2/2     Running   0          4m41s
...
```

- Create a default policy to deny all ingress traffic

```
ubuntu@k8smaster:~/secruty$ cat deny-all.yaml 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

- an example of NetworkPolicy

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:        # Only pod with role=db label will be included
    matchLabels:
      role: db
  policyTypes:        # both Ingress and Egress policies are exist
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16   # can receive traffic from this IP range
        except:               # only except this range
        - 172.17.1.0/24
    - namespaceSelector:      # can receive traffic from namespace with this label
        matchLabels:
          project: myproject
    - podSelector:            # can receive traffic from pod with role=frontend
        matchLabels:
          role: frontend
    ports:                    # only can get traffic from port 6379 via TCP 
    - protocol: TCP
      port: 6379
  egress:
  - to:                       # only can send data to this IP range
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:                    # data can only be sent by this port via TCP
    - protocol: TCP
      port: 5978
```

- refer

[Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
