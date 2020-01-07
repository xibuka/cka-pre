## Know various ways to configure applications

- Command Line arguments to container

```
kind: Pod
spec:
  containers:
  - image: some/image
    command: ["/bin/command"]
    args: ["arg1", "arg2", "arg3"]
```

- environment variables

```
kind: Pod
spec:
  containers:
  - image: some/image
    env:
    - name: INTERVAL
      value: "30"
    name: html-generator
```

refer to other environment variables
```
env:
- name: FIRST_VAR
  value: "foo"
- name: SECOND_VAR
  value: "$(FIRST_VAR)bar"
```

- ConfigMap

  - Create Configmap from command line

```
ubuntu@k8smaster:~/application$ kubectl create configmap myconfig \
>       --from-literal=foo=bar --from-literal=bar=baz --from-literal=one=two
configmap/myconfig created

ubuntu@k8smaster:~/application$ kubectl get cm myconfig -o yaml
apiVersion: v1
data:
  bar: baz
  foo: bar
  one: two
kind: ConfigMap
metadata:
  creationTimestamp: "2020-01-07T08:49:33Z"
  name: myconfig
  namespace: default
  resourceVersion: "132436"
  selfLink: /api/v1/namespaces/default/configmaps/myconfig
  uid: 8fe509d6-3524-4eec-97d7-a3aed16e91f3
```

  - Create Configmap from yaml file
```
ubuntu@k8smaster:~/application$ cat configmap.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfig
data:
  bar: baz
  foo: bar
  one: two

ubuntu@k8smaster:~/application$ kubectl create -f configmap.yaml 
configmap/myconfig created
ubuntu@k8smaster:~/application$ kubectl get cm myconfig -o yaml
apiVersion: v1
data:
  bar: baz
  foo: bar
  one: two
kind: ConfigMap
metadata:
  creationTimestamp: "2020-01-07T08:52:22Z"
  name: myconfig
  namespace: default
  resourceVersion: "132843"
  selfLink: /api/v1/namespaces/default/configmaps/myconfig
  uid: 32e0f8c2-78b3-4328-a420-de6532b49efe
```

- Pass Configmap as Environment variables to Pod

Create configmap
```
ubuntu@k8smaster:~$ kubectl create configmap special-config --from-literal=special.how=very
configmap/special-config created
```

create a pod which use this configmap's value as environment value
```
ubuntu@k8smaster:~/application$ cat pod-cm-env.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      env:
        # Define the environment variable
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: special-config
              # Specify the key associated with the value
              key: special.how
  restartPolicy: Never
ubuntu@k8smaster:~/application$ kubectl create -f pod-cm-env.yaml 
pod/dapi-test-pod created
ubuntu@k8smaster:~/application$ kubectl logs dapi-test-pod | grep very
SPECIAL_LEVEL_KEY=very
```

- Pass Configmap as a volume to pod

```
ubuntu@k8smaster:~/application$ cat configmap-volume-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
  - name: app-container
    image: busybox
    command: ["cat", "/etc/config/special.how"]
    volumeMounts:
      - name: configmapvolume
        mountPath: /etc/config
  volumes:
    - name: configmapvolume
      configMap:
        name: special-config
ubuntu@k8smaster:~/application$ kubectl apply -f configmap-volume-pod.yaml 
pod/configmap-volume-pod created

ubuntu@k8smaster:~/application$ kubectl get pod
NAME                   READY   STATUS      RESTARTS   AGE
configmap-volume-pod   0/1     Completed   0          7s
ubuntu@k8smaster:~/application$ kubectl logs configmap-volume-pod
very
```

- create secret

```
ubuntu@k8smaster:~/application$ cat secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: appsecret
stringData:
  cert: value
  key: value
ubuntu@k8smaster:~/application$ kubectl create -f secret.yaml 
secret/appsecret created
ubuntu@k8smaster:~/application$ kubectl get secrets 
NAME                  TYPE                                  DATA   AGE
appsecret             Opaque                                2      5s
```

- pass secret as Environment variables

```
ubuntu@k8smaster:~/application$ cat secret-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app-container
    image: busybox
    command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]
    env:
    - name: MY_CERT
      valueFrom:
        secretKeyRef:
          name: appsecret
          key: cert
ubuntu@k8smaster:~/application$ kubectl create -f secret-env.yaml
pod/secret-pod created

ubuntu@k8smaster:~/application$ kubectl exec -it secret-pod -- sh
/ # echo $MY_CERT
value

```

- pass secret as volume

```
ubuntu@k8smaster:~/application$ cat secret-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
  - name: app-container
    image: busybox
    command: ['sh', '-c', "echo  && sleep 3600"]
    volumeMounts:
      - name: secretvolume
        mountPath: /etc/certs
  volumes:
    - name: secretvolume
      secret:
        secretName: appsecret
ubuntu@k8smaster:~/application$
ubuntu@k8smaster:~/application$ kubectl create -f secret-volume.yaml
pod/secret-volume-pod created
ubuntu@k8smaster:~/application$ kubectl exec secret-volume-pod -- ls /etc/certs
cert
key
ubuntu@k8smaster:~/application$ kubectl exec secret-volume-pod -- cat /etc/certs/cert
value
```

- refer

[config configmap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

[secret](https://kubernetes.io/docs/concepts/configuration/secret/)

[environment variable](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)
