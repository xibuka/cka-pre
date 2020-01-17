## Understand Access Modes for volumes

- Access Mode

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: d-bp1j17ifxfasvts3tf40
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /tmp
    server: 172.17.0.2
```

    ReadWriteOnce – the volume can be mounted as read-write by a single node
    ReadOnlyMany – the volume can be mounted read-only by many nodes
    ReadWriteMany – the volume can be mounted as read-write by many nodes

a PV can have multi types of mode, but for PVC only 1 mode is available

- refer
[access mode](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)
