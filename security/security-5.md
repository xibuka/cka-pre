## Work with images securely

Now Assumption we have a image in a private registry

- create a secret to access private registry

```
kubectl create secret docker-registry acr --docker-server=https://URL --docker-username=username --docker-password='password' --docker-email=user@example.com
```

- use this secret in the pod
```
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: acr
```

- use this secret in the namespace

```
kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "acr"}]}'
```

- refer

[Add ImagePullSecrets to a service account](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account)

[Pull an Image from a Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

