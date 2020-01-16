## Create and manage TLS certificates for cluster components

- The CA certificate on a pod in your cluster
```
kubectl exec busybox -- ls /var/run/secrets/kubernetes.io/serviceaccount
```

- generate a CSR 

Download the cfssl(no need in real exam)

```
ubuntu@k8smaster:~/secruty$ wget -q --show-progress --https-only --timestamping \
>   https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 \
>   https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
cfssl_linux-amd64                             100%[================================================================================================>]   9.90M  5.39MB/s    in 1.8s    
cfssljson_linux-amd64                         100%[================================================================================================>]   2.17M  2.27MB/s    in 1.0s    
ubuntu@k8smaster:~/secruty$ chmod +x cfssl_linux-amd64 cfssljson_linux-amd64
ubuntu@k8smaster:~/secruty$ sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
ubuntu@k8smaster:~/secruty$ sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
ubuntu@k8smaster:~/secruty$ cfssl version
Version: 1.2.0
Revision: dev
Runtime: go1.6
```

create a CSR file:
```
ubuntu@k8smaster:~/secruty$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   46h
nginx        ClusterIP   10.96.25.7   <none>        80/TCP    56m
ubuntu@k8smaster:~/secruty$ kubectl get pod -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
nginx-555786ff8-5b86v   1/1     Running   2          32h   10.244.2.43   k8snode2    <none>           <none>
nginx-555786ff8-6lv8p   1/1     Running   2          32h   10.244.2.42   k8snode2    <none>           <none>
nginx-555786ff8-9gv5d   1/1     Running   2          32h   10.244.0.60   k8smaster   <none>           <none>
nginx-555786ff8-jw5f9   1/1     Running   2          32h   10.244.0.58   k8smaster   <none>           <none>
nginx-555786ff8-smcjd   1/1     Running   2          32h   10.244.0.59   k8smaster   <none>           <none>
ubuntu@k8smaster:~/secruty$ cat <<EOF | cfssl genkey - | cfssljson -bare server
> {
>   "hosts": [
>     "nginx.default.svc.cluster.local",
>     "nginx-555786ff8-5b86v.default.pod.cluster.local",
>     "10.96.25.7",
>     "10.244.2.43"
>   ],
>   "CN": "nginx-555786ff8-5b86v.default.pod.cluster.local",
>   "key": {
>     "algo": "ecdsa",
>     "size": 256
>   }
> }
> EOF
2020/01/09 14:34:07 [INFO] generate received request
2020/01/09 14:34:07 [INFO] received CSR
2020/01/09 14:34:07 [INFO] generating key: ecdsa-256
2020/01/09 14:34:07 [INFO] encoded CSR

ubuntu@k8smaster:~/secruty$ ls
canal.yaml  deny-all.yaml  server-key.pem  server.csr
ubuntu@k8smaster:~/secruty$ cat <<EOF | kubectl create -f -
> apiVersion: certificates.k8s.io/v1beta1
> kind: CertificateSigningRequest
> metadata:
>   name: pod-csr.web
> spec:
>   groups:
>   - system:authenticated
>   request: $(cat server.csr | base64 | tr -d '\n')
>   usages:
>   - digital signature
>   - key encipherment
>   - server auth
> EOF
certificatesigningrequest.certificates.k8s.io/pod-csr.web created
ubuntu@k8smaster:~/secruty$
ubuntu@k8smaster:~/secruty$ kubectl get csr
NAME          AGE   REQUESTOR          CONDITION
pod-csr.web   9s    kubernetes-admin   Pending
```

- Approve the CSR
```
ubuntu@k8smaster:~/secruty$  kubectl certificate approve pod-csr.web
certificatesigningrequest.certificates.k8s.io/pod-csr.web approved

ubuntu@k8smaster:~/secruty$ kubectl get csr
NAME          AGE   REQUESTOR          CONDITION
pod-csr.web   89s   kubernetes-admin   Approved,Issued
```

Now you can use `server.crt` and `server-key.pem` as the keypair to start your
HTTPS server.

- refer

[Manage TLS Certificates in a Cluster](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/)
