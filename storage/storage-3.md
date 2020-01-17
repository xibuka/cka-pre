## Understand persistent volume claims primitive

- Create a pvc

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  persistentVolumeReclaimPolicy: Retain
  resources:
    requests:
      storage: 8Gi
  storageClassName: ""
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```

**persistentVolumeReclaimPolicy:**

	Retain: The data in pv will not be touched automatically, need admin to
            deal with it.
	Recycle: Reuse pv, will execute `rm -rf /thevolume/*` Departed
	Delete: Delete the pv, the default mode for dynamic provision

**AccessMode**: same with PV

**volumeMode**: the file system of volume, same with pv. 

**resources**: defination about the requested resource

**selector**: only pv which match the selector will be bound to pvc.

**storageClassName**: specify the pv's storageClass

- use pvc in pod

pod and pvc must in the same namespace.

```
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: dockerfile/nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```


