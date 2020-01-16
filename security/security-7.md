## Secure persistent key value store

- Confirm secret
```
ubuntu@k8smaster:~$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
appsecret             Opaque                                2      8d
default-token-g4zrc   kubernetes.io/service-account-token   3      21d
jenkins-token-d5fwr   kubernetes.io/service-account-token   3      6d21h
ubuntu@k8smaster:~$ kubectl describe secrets default-token-g4zrc
Name:         default-token-g4zrc
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: e8705f53-3281-4087-9df7-f04b252554e9

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IlF3cEdRS24tdDF0b3dWMEw5OURxNUtzczRFLVFuX2F1OHdFVERldjlPZ28ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tZzR6cmMiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImU4NzA1ZjUzLTMyODEtNDA4Ny05ZGY3LWYwNGIyNTI1NTRlOSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.meIMFJZ7-W6DFbwcuMVvcof4u3EyPyWKTCUvjkIZ02ClhQnIIauw9-twkBgt3sz9EpXunBKVzsnR2qv8EoaXnYfba4wInvWBxHvNlswxOkdeGN5V8Cq6TuXdoKgVKw2YmFVTdQaBK0vEUlytanjC767RSMhTiH5VWymPj6kbwgm3vFsdEW5FG0ymisNDp_Xn6FiPzu_N4QcuSUm_0K-yMsWztyyqN7EgWnn6YaiJpNGpa2kmqcXy9s3K5bobiDLz0DA8_8rw4GNPWWysAPzwMHwJvcpLAs6FaBof-Ke15_fIbObOjLB3ecHatVpmT2mINpZu3hmwhDYIlCVeDqwTtA
ca.crt:     1025 bytes
namespace:  7 bytes

```

- confirm the default secret is used in the pod

```
ubuntu@k8smaster:~$ kubectl describe pod nginx
Name:         nginx
Namespace:    default
Priority:     0
Node:         k8snode1/20.0.1.55
Start Time:   Thu, 16 Jan 2020 05:42:21 +0000
Labels:       run=nginx
Annotations:  cni.projectcalico.org/podIP: 10.244.1.6/32
Status:       Running
IP:           10.244.1.6
IPs:
  IP:  10.244.1.6
Containers:
  nginx:
    Container ID:   docker://e7099a96d40a291c1dee6817a2233632d20739fcdc82401b93d52be304cc013b
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:8aa7f6a9585d908a63e5e418dc5d14ae7467d2e36e1ab4f0d8f9d059a3d071ce
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 16 Jan 2020 05:42:30 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-g4zrc (ro)
...
```

- create a secret for https usage

```
ubuntu@k8smaster:~$ openssl genrsa -out https.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
...........+++++
......................+++++
e is 65537 (0x010001)
ubuntu@k8smaster:~$ openssl req -new -x509 -key https.key -out https.cert -days=365 -subj /CN=www.example.com
Can't load /home/ubuntu/.rnd into RNG
140635010126272:error:2406F079:random number generator:RAND_load_file:Cannot open file:../crypto/rand/randfile.c:88:Filename=/home/ubuntu/.rnd
ubuntu@k8smaster:~$ 
ubuntu@k8smaster:~$ ls https.cert 
https.cert
ubuntu@k8smaster:~$ touch file
ubuntu@k8smaster:~$ ls https.* file -l
-rw-rw-r-- 1 ubuntu ubuntu    0 Jan 16 05:46 file
-rw-rw-r-- 1 ubuntu ubuntu 1131 Jan 16 05:46 https.cert
-rw------- 1 ubuntu ubuntu 1679 Jan 16 05:45 https.key
```

- create the secret
```
ubuntu@k8smaster:~$ kubectl create secret generic example-https --from-file=https.cert --from-file=https.key --from-file=file
secret/example-https created
ubuntu@k8smaster:~$
ubuntu@k8smaster:~$ kubectl get secret
NAME                  TYPE                                  DATA   AGE
appsecret             Opaque                                2      8d
default-token-g4zrc   kubernetes.io/service-account-token   3      21d
example-https         Opaque                                3      14s
jenkins-token-d5fwr   kubernetes.io/service-account-token   3      6d21h
ubuntu@k8smaster:~$ kubectl get secret example -o yaml
Error from server (NotFound): secrets "example" not found
ubuntu@k8smaster:~$ kubectl get secret example-https -o yaml
apiVersion: v1
data:
  file: ""
  https.cert: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURGVENDQWYyZ0F3SUJBZ0lVSjBGdWMwYWhRa2tiZlArVFkwVGxZMEhNaFNBd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0dqRVlNQllHQTFVRUF3d1BkM2QzTG1WNFlXMXdiR1V1WTI5dE1CNFhEVEl3TURFeE5qQTFORFl3TmxvWApEVEl4TURFeE5UQTFORFl3Tmxvd0dqRVlNQllHQTFVRUF3d1BkM2QzTG1WNFlXMXdiR1V1WTI5dE1JSUJJakFOCkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQTJFYTF0V1lKQUtWblhZS3lRTGZpM3BpV3ZHd1QKam82ZkpsRlZDZWNCc25FeGNQNEhwVUNValdpVEtQSnZ2eTAvVzRHSXBnNEwvTlpQNWtMc3BOU21DcVVZeTFjMApESzFiMjdudXppT2dVZHNJOVJnREJORis1Um5idUZtVUdPNWFyYnJ1Y0krWnpRTGJyNkpLQ2hmZ3h2ais3cU1RClVKaWhEVTZUbDZOTTNhZEpIU1dRWGh3aWRqTXVSNFFYZnExN0hvbHF6dEtIZS8zWmwwWlNOQ1k2bGdaUCsrOHUKeHZmSHUwY3FDMFdzZ1JqLzhFK0Fid3ZnOWQzY0JOUzBmMW5EeittenZWTEVkdS92aGZLeGsrcDlqcXJkN2MxdwpURXhvdWNLZGtOWjAxQU1wZHZOclp3QVd1N1R1SldtREtjWnJFcDA0TVhZSjR4ZTM4bk1lUkU4VFpRSURBUUFCCm8xTXdVVEFkQmdOVkhRNEVGZ1FVNXRVdTc5d2JDaWN0Q1BZRVB0K2Jyb2lRS3Awd0h3WURWUjBqQkJnd0ZvQVUKNXRVdTc5d2JDaWN0Q1BZRVB0K2Jyb2lRS3Awd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBTkJna3Foa2lHOXcwQgpBUXNGQUFPQ0FRRUFaK2w0bWVJWktvcFZnWDB1V1ZiVWgyVThFTmVzTFZFbDNCa1ZJVGsyMldNTlljcStCckhuCmw1UUlEd2ZMTDF1aDdjM01pMm5Db1RwOEg5cU5RQmdJT1lrQW1MYWZidUg0ZGNzWEhIZGVLM0NVL1V2OVdDckwKK2ZsbXVMdC90SEVVRk1mMGZ3VGFhekgvOVBHeEFhOW1UQUc0Z3h5SzBrSEZRMWhBMmZieVFTc3l5ZjlHNjVUUwpoWldNS08yNEJydWV6SzFkWEhYWmo2S1FNSnJPVUZibkQ1YTdRbkcySTdWT3lzNG4yYUZNUkU0Kzd4NytpM0c0CkhVK0FDeVJOSGc4QTFqd2svb3VxdFJyMmVSajlIamZ4ZkM5Z2lzb0hKa1I4ZzZJY0VDTkFFTVU1eWc4OXQxVXAKMS9ZU0x1MlFtRy9kQ2JyRmtGSEJ2TnlSOUVGQ0NGRmhOQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  https.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBMkVhMXRXWUpBS1ZuWFlLeVFMZmkzcGlXdkd3VGpvNmZKbEZWQ2VjQnNuRXhjUDRICnBVQ1VqV2lUS1BKdnZ5MC9XNEdJcGc0TC9OWlA1a0xzcE5TbUNxVVl5MWMwREsxYjI3bnV6aU9nVWRzSTlSZ0QKQk5GKzVSbmJ1Rm1VR081YXJicnVjSStaelFMYnI2SktDaGZneHZqKzdxTVFVSmloRFU2VGw2Tk0zYWRKSFNXUQpYaHdpZGpNdVI0UVhmcTE3SG9scXp0S0hlLzNabDBaU05DWTZsZ1pQKys4dXh2Zkh1MGNxQzBXc2dSai84RStBCmJ3dmc5ZDNjQk5TMGYxbkR6K216dlZMRWR1L3ZoZkt4aytwOWpxcmQ3YzF3VEV4b3VjS2RrTlowMUFNcGR2TnIKWndBV3U3VHVKV21ES2NackVwMDRNWFlKNHhlMzhuTWVSRThUWlFJREFRQUJBb0lCQUd4dTkwZWRnc0g2SzlycwpYaWFvaTJ5RElJQVR4N0pmdTFkZ3k3d05RQUtSVWZLT3lwS0YwSFVkaXpxcVluQjlaUmloMXpzNks3UjJWdFRoCmxQZ0hUY0JraWd2WkN0V1lUVWZhN3VvWFhwZnJzNC8rbS9UY2ZEeXRQRVl2VTFzMGxlNG9uTWQrbCtQN25OMXcKQnFNTTJidW90MWc2RlVIelpEdmh1NG41YTk1c3hyaVNkVWQxN2RFKzFudGZ6MGovKzdnY0pCcW5ROWc2ZTFxaQpXb3MxeU50MjJwaFBLOWRDRzdXZU5CR2lhLzJZMmxMWG9tcjJRVm42d0JKb1ZXVDhMa0d5azRaTnNSNWFXOGNLClFiTzBTTTlyNWFBeUtuWnZVa3VmME9vOXBwSzZwQ3A5eUl2MVFqYU5ZdmoxeGtiYXJKa29iSnlvNElFM0FXV2sKR1ZLbkVNRUNnWUVBOXU5T1VVdGVQU3FsL1ByekRMRUhuZTBRdkhlSVpnMVltcXl0cjJ6RlhVc1I0bW0xSGVNbwpjTmRJQm51a29uY3lPQkNiUHJidyt4RmVURUo2UUx0Wlc3bW9aWm13bVFTYlpyUzZXWGJHaXk5Y0kxbHJvR0dJCksrWEpiVWF0b3hkTFArRmlyOUsvcUtHUFI0RXNjdWdpbVdWeDNveEo0L2lWUVV0cmxUeDVxSWtDZ1lFQTREZEcKUVZGVGVaRHpGak15VEJvZGo1ZlkralJiOCtlcmYxc09KY3dnM2lmWnNFRURvdHhRdGYydFFjSzRJb3loTFBrNgpaT0l2OW5SaU1FOG1wSmNJMzZadVJCVHlFRzA5S1AzTTdpZTRpTjRVTGFkSThCYnhtM2VKOVd0d1NkdC8xemcyCmVQZWE5aHJsN0daKysyOEdBYnUzRTl3U2taS2hPOHNmS256T1pQMENnWUVBbFFUcDRJbDVQN1NESTE1V1d1eGkKeWwyTVloQkkwajF2b1RoZ1FLT0ZuNzF6OTQwUGJnL3VFZHI1Ym1BamhLQW1RRXRWUk0ybU0vM1JTSGc5eXQ3RgpHR084U2tRcm5NeDQ4OHhSUVRnNnJUaFJoRXVzZGNjbUpFZXgvUzVRRDBJNWVUMk5Ec3BDTzRQME1aUzB2RXQyCnhkZkFsaXRYVkNwcCtGT0pnekJSd3ZFQ2dZRUFzUkYxM2llSHNMMlQrN2c2eEhicldYY2wxNUo1KzhpOVd6cGgKbGQvN0pQWjdxQUh3Q1RITVc2MFdvcFJRTHBpNHdIZWljZ0ZldDFkNkk0U2VrK1RqRVJ5eWYvbTZvTlprTW5jYwozQWRxYUV4Wnl5UU5LZTQwcC81amFQbU1HQWZNa2Y3R3BnbUV5MDY2dlZMRWZYUlVYaElNcHhacFk2VlV4NC9GCjhSdlhNMUVDZ1lBNng2RlFpNjl4L1hJMkpqeVlJWmppWDJkZjZ1Mm1TU01TbnhQY0NTZWN5WDQ5aDBqWjJla2MKL3N3SURNM1paRDJBUDJZb3NyZXJSM1N3QThkY2l2TzBtSTM0ZllRNmNaSS8wUmI1dFNCUkNtVWhGcE91dVlBRgpXUFZFREF6UkxnYWsxdVl2d3orNVhQVjNTUEl5UGFER2kxOHRXNUoxUlp4M1YxaURaU2doTEE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
kind: Secret
metadata:
  creationTimestamp: "2020-01-16T05:55:02Z"
  name: example-https
  namespace: default
  resourceVersion: "288930"
  selfLink: /api/v1/namespaces/default/secrets/example-https
  uid: a0f0d11d-6d59-48fb-9fec-1850d5601892
type: Opaque
```

- mound the secret in the pod

- use the secret to enable https in pod

```

```

```
ubuntu@k8smaster:~/secruty$ vim config.cm
ubuntu@k8smaster:~/secruty$
ubuntu@k8smaster:~/secruty$ vim example-https.yaml
ubuntu@k8smaster:~/secruty$ mv config.cm config.yaml
ubuntu@k8smaster:~/secruty$
ubuntu@k8smaster:~/secruty$ ls
canal.yaml  config.yaml  deny-all.yaml  example-https.yaml  file  https.cert  https.key  server-key.pem  server.csr
ubuntu@k8smaster:~/secruty$ kubectl create -f config.yaml
configmap/config created
ubuntu@k8smaster:~/secruty$ kubectl create -f example-https.yaml
pod/example-https created
ubuntu@k8smaster:~/secruty$ kubectl get pod
NAME            READY   STATUS    RESTARTS   AGE
example-https   2/2     Running   0          35s
nginx           1/1     Running   0          17m
ubuntu@k8smaster:~/secruty$ kubectl exec example-https -c web-server -- mount  | grep certs
tmpfs on /etc/nginx/certs type tmpfs (ro,relatime)
ubuntu@k8smaster:~/secruty$ kubectl port-forward example-https 8443:443 &
[1] 5205

ubuntu@k8smaster:~/secruty$ Forwarding from 127.0.0.1:8443 -> 443
Forwarding from [::1]:8443 -> 443

ubuntu@k8smaster:~/secruty$ curl https://localhost:8443 -k
Handling connection for 8443
So she went into the garden to cut a cabbage leaf to make an apple pie;
and at the same time a great she-bear, coming up the street pops its head
into the shop. "What! no soap?" So he died, and she very imprudently
married the barber; and there were present the Picninnies, and the Grand
Panjandrum himself, with the little round button at top, and they all
fell to playing the game of catch as catch can, till the gunpowder ran
out at the heels of their boots.
		-- Samuel Foote
```


- Use the secret in the pod as files
```
apiVersion: v1
kind: Pod
metadata:
  name: fortune-https
spec:
  containers:
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: certs
      mountPath: /etc/nginx/certs/
      readOnly: true
volumes:
- name: certs
  secret:
    secretName: fortune-https
```

- Use the secret in the pod to access a private image registry
```
apiVersion: v1
kind: Pod
metadata:
  name: private-pod
spec:
  imagePullSecrets:
  - name: mydockersecret
  containers:
  - image: username/private:tag
    name: main
```

- refer

[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
