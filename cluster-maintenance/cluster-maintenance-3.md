## Implement backup and restore methodologies

All data of kubernetes cluster are saved in etcd, so only backup etcd is
enough.


- download etcd binaries

```
ubuntu@k8sm1:~$ wget https://github.com/etcd-io/etcd/releases/download/v3.3.12/etcd-v3.3.12-linux-amd64.tar.gz
...

2020-01-08 08:31:18 (368 KB/s) - ‘etcd-v3.3.12-linux-amd64.tar.gz’ saved [11350736/11350736]

ubuntu@k8sm1:~$ tar xvf etcd-v3.3.12-linux-amd64.tar.gz
ubuntu@k8sm1:~$ sudo mv etcd-v3.3.12-linux-amd64/etcd* /usr/local/bin
```

- backup

```
ETCDCTL_API=3 etcdctl snapshot save mysnapshot.db     \
           --endpoints=https://127.0.0.1:2379         \
           --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
           --cert=/etc/kubernetes/pki/etcd/server.crt \
           --key=/etc/kubernetes/pki/etcd/server.key
```

- restore

```
ETCDCTL_API=3 etcdctl snapshot restore snapshotdb     \
           --endpoints=https://127.0.0.1:2379         \
           --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
           --cert=/etc/kubernetes/pki/etcd/server.crt \
           --key=/etc/kubernetes/pki/etcd/server.key
```

- refer

[Backing Up an etcd cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)
