## Understand persistent volumes and know how to create them;

2 types of resources to provide persistent volumes
  - PersistentVolume (PV) : Create storage resource from underlayer resources.
  - PersistentVolumeClaim (PVC) : Create an object which can be used in POD

- Create a PV

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-pv
  labels:
    failure-domain.beta.kubernetes.io/zone: cn-hangzhou-c
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: Manual
  persistentVolumeReclaimPolicy: Delete
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

**storageClassName**: 
     Name of StorageClass to which this persistent volume belongs. Empty value
     means that this volume does not belong to any StorageClass.

**mountOptions**: 
     A list of mount options, e.g. ["ro", "soft"]. Not validated - mount will
     simply fail if one is invalid.

- Create a PVC

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-pvc
  labels:
    failure-domain.beta.kubernetes.io/zone: cn-hangzhou-c
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  storageClassName: alicloud-disk-common-zone
  resources:
    requests:
      storage: 20Gi
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```

**volumeMode**
     volumeMode defines what type of volume is required by the claim. Value of
     Filesystem is implied when not included in claim spec. This is a beta
     feature.

**selector**
     A label query over volumes to consider for binding.

     A label selector is a label query over a set of resources. The result of
     matchLabels and matchExpressions are ANDed. An empty label selector matches
     all objects. A null label selector matches no objects.

**storageClassName**
     Name of the StorageClass required by the claim. 

- Bind PV and PVC

Dynamic
    PVC will always be binded with the PV which is created by a Provisioner.

Static
	accessModes: totally match
	storage: must large than request
	label: totally match
	storageClassName totally match

- refer
[Persistent Volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
