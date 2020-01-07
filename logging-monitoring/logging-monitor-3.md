## Manage cluster component logs


- the container logs in node
```
ubuntu@k8smaster:~$ sudo ls /var/log/containers/
etcd-k8smaster_kube-system_etcd-3c88f905f5eb52d3ca33b2a8c30249a84159e3dc10f7b372bacc9ba819ecf8fa.log
etcd-k8smaster_kube-system_etcd-c7fa61b8e99f5acd80973a96dad48a24ca28698ffea9c1d4b34bec3ffea0d34d.log
kube-apiserver-k8smaster_kube-system_kube-apiserver-833ebd7541d53bf85d7a57a035f6f9f9ad6887d6b02690e12dc481b2623a8b56.log
kube-apiserver-k8smaster_kube-system_kube-apiserver-a3cbe7255c3e531301339579598dc1d66f64a25ebbf89e38bc5e307dad537b47.log
kube-controller-manager-k8smaster_kube-system_kube-controller-manager-8ea3f3f79277279c836559b5c55e58b3724bc763025bd55a79c1f42b6d7312a6.log
kube-controller-manager-k8smaster_kube-system_kube-controller-manager-cf78b95817ef4ecb862456dbfca287d72faf99d6ec8af1594e7502e0317930b9.log
kube-flannel-ds-amd64-bfg88_kube-system_install-cni-3beb52ceccab235d517338f5718a946c5e4c465a16335306658929d3e79b9360.log
kube-flannel-ds-amd64-bfg88_kube-system_kube-flannel-26cdf1cf9b5a37f03a4542423eeed51154bb137916f225b6b9ddd3a59847a72f.log
kube-flannel-ds-amd64-bfg88_kube-system_kube-flannel-5cd91b33a039304f311875280aea3cc75c190b3ee8a230833ad545ffe930055b.log
kube-proxy-w4xsz_kube-system_kube-proxy-08cd7f5e401b95e7076e20cc7c98e881145d228e225628802cba99c6e1f21510.log
kube-proxy-w4xsz_kube-system_kube-proxy-831521aa1b39343e038f629b14a7765a252a8f57f9ed708fc6b77848b16fd455.log
kube-scheduler-k8smaster_kube-system_kube-scheduler-8c62b87a74cde7263de2a3dfdccd6e80df006b41a40d3607386890e0c9c8da84.log
kube-scheduler-k8smaster_kube-system_kube-scheduler-ea6f7c5ca09b599989baf4e8599c7109a7303dfd0abfe5f8b321045e5360b5fb.log
nginx-readiness-6bcfbd848-8vcr5_default_nginx-386b99e69d71a65e034dca7c12c858a68a008f7831cde7f64cfea360cafdb344.log
```

- the kubelet log
```
/var/log/syslog
```

- the container logs in the pod

```
ubuntu@k8smaster:~$ cat sidecar.yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-log-1
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/1.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-log-2
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/2.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}
```

Above Pod has 2 containers

```
ubuntu@k8smaster:~$ kubectl logs counter count-log-1 
0: Tue Jan  7 05:25:44 UTC 2020
1: Tue Jan  7 05:25:45 UTC 2020
2: Tue Jan  7 05:25:46 UTC 2020
...
ubuntu@k8smaster:~$ kubectl logs counter count-log-2
Tue Jan  7 05:25:44 UTC 2020 INFO 0
Tue Jan  7 05:25:45 UTC 2020 INFO 1
Tue Jan  7 05:25:46 UTC 2020 INFO 2
...

```

- refer
[logging](https://kubernetes.io/docs/concepts/cluster-administration/logging/)
