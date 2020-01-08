## Understand Kubernets cluster upgrade process

- confirm the version

`kubectl get nodes` is the version of `kubelet`
```
NAME    STATUS   ROLES    AGE   VERSION
k8sm1   Ready    master   37m   v1.13.8
k8sm2   Ready    <none>   35m   v1.13.8
```

`kubectl version --short` is the version of `kubectl` and `api server`
```
ubuntu@k8sm1:~$ kubectl version --short
Client Version: v1.13.8
Server Version: v1.13.12
```

`kubectl describe nodes` can see the version of `kubelet`, `docker`,
`kube-Proxy`.

```
ubuntu@k8sm1:~$ kubectl describe node | grep Version
 Kernel Version:             4.15.0-74-generic
 Container Runtime Version:  docker://19.3.4
 Kubelet Version:            v1.13.8
 Kube-Proxy Version:         v1.13.8
 Kernel Version:             4.15.0-74-generic
 Container Runtime Version:  docker://19.3.4
 Kubelet Version:            v1.13.8
 Kube-Proxy Version:         v1.13.8
```

The controller version can be seen via the pod describe

```
ubuntu@k8sm1:~$ kubectl get pod -n kube-system | grep controller
kube-controller-manager-k8sm1   1/1     Running   0          37m
ubuntu@k8sm1:~$ kubectl describe pod kube-controller-manager-k8sm1 -n kube-system| grep Image
    Image:         k8s.gcr.io/kube-controller-manager:v1.13.12
    Image ID:      docker-pullable://k8s.gcr.io/kube-controller-manager@sha256:cd8f801a7c258ec34d81cd609f442e82dbba36fc226becbbea096a66b74e7044
```

- Upgrade kubeadm

```
ubuntu@k8sm1:~$ sudo apt-mark unhold kubeadm                                        
Canceled hold on kubeadm.
ubuntu@k8sm1:~$ sudo apt install kubeadm=1.14.1-00                                         
Reading package lists... Done
...
Preparing to unpack .../kubeadm_1.14.1-00_amd64.deb ...
Unpacking kubeadm (1.14.1-00) over (1.13.8-00) ...
Setting up kubeadm (1.14.1-00) ...
ubuntu@k8sm1:~$ sudo apt-mark hold kubeadm
kubeadm set on hold.
ubuntu@k8sm1:~$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.1", GitCommit:"b7394102d6ef778017f2ca4046abbaa23b88c290", GitTreeState:"clean", BuildDate:"2019-04-08T17:08:49Z", GoVersion:"go1.12.1", Compiler:"gc", Platform:"linux/amd64"}
```

- Upgrade kubernetes controller component
```
ubuntu@k8sm1:~$ sudo kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade] Making sure the cluster is healthy:
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.13.12
[upgrade/versions] kubeadm version: v1.14.1
I0108 05:22:18.119418   19466 version.go:240] remote version is much newer: v1.17.0; falling back to: stable-1.14
[upgrade/versions] Latest stable version: v1.14.10
[upgrade/versions] Latest version in the v1.13 series: v1.13.12

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       AVAILABLE
Kubelet     2 x v1.13.8   v1.14.10

Upgrade to the latest stable version:

COMPONENT            CURRENT    AVAILABLE
API Server           v1.13.12   v1.14.10
Controller Manager   v1.13.12   v1.14.10
Scheduler            v1.13.12   v1.14.10
Kube Proxy           v1.13.12   v1.14.10
CoreDNS              1.2.6      1.3.1
Etcd                 3.2.24     3.3.10

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.14.10

Note: Before you can perform this upgrade, you have to update kubeadm to v1.14.10.

_____________________________________________________________________

ubuntu@k8sm1:~$ 
ubuntu@k8sm1:~$ sudo kubeadm upgrade apply v1.14.1
[preflight] Running pre-flight checks.
[upgrade] Making sure the cluster is healthy:
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[upgrade/version] You have chosen to change the cluster version to "v1.14.1"
[upgrade/versions] Cluster version: v1.13.12
[upgrade/versions] kubeadm version: v1.14.1
[upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
[upgrade/prepull] Will prepull images for components [kube-apiserver kube-controller-manager kube-scheduler etcd]
[upgrade/prepull] Prepulling image for component etcd.
[upgrade/prepull] Prepulling image for component kube-apiserver.
[upgrade/prepull] Prepulling image for component kube-controller-manager.
[upgrade/prepull] Prepulling image for component kube-scheduler.
[apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-etcd
[apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-controller-manager
[apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-scheduler
[apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-apiserver
[apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-controller-manager
[apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-etcd
[apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-apiserver
[apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-scheduler
[upgrade/prepull] Prepulled image for component kube-apiserver.
[upgrade/prepull] Prepulled image for component kube-controller-manager.
[upgrade/prepull] Prepulled image for component kube-scheduler.
[upgrade/prepull] Prepulled image for component etcd.
[upgrade/prepull] Successfully prepulled the images for all the control plane components
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.14.1"...
Static pod: kube-apiserver-k8sm1 hash: 4ee5be0bc66717b17b7e2a12aa2e2e76
Static pod: kube-controller-manager-k8sm1 hash: fb359d5eea55c56290c8b6cc8245ed82
Static pod: kube-scheduler-k8sm1 hash: 97bd2ffa08a518ac4f55aa68c229d9f1
[upgrade/etcd] Upgrading to TLS for etcd
Static pod: etcd-k8sm1 hash: e1aa8c91f5d8648fa1fdd9359e40db2b
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/etcd.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-01-08-05-25-00/etcd.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
Static pod: etcd-k8sm1 hash: e1aa8c91f5d8648fa1fdd9359e40db2b
Static pod: etcd-k8sm1 hash: 17fc0f882c17341f56aed655e8d06c41
[apiclient] Found 1 Pods for label selector component=etcd
[upgrade/staticpods] Component "etcd" upgraded successfully!
[upgrade/etcd] Waiting for etcd to become available
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests139186667"
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-01-08-05-25-00/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
Static pod: kube-apiserver-k8sm1 hash: 4ee5be0bc66717b17b7e2a12aa2e2e76
Static pod: kube-apiserver-k8sm1 hash: 4ee5be0bc66717b17b7e2a12aa2e2e76
Static pod: kube-apiserver-k8sm1 hash: 8dd4b186cf82680e13f466a049ad8241
[apiclient] Found 1 Pods for label selector component=kube-apiserver
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-01-08-05-25-00/kube-controller-manager.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
Static pod: kube-controller-manager-k8sm1 hash: fb359d5eea55c56290c8b6cc8245ed82
Static pod: kube-controller-manager-k8sm1 hash: b79360c88b3648b0201873a2b7f946e6
[apiclient] Found 1 Pods for label selector component=kube-controller-manager
[upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-01-08-05-25-00/kube-scheduler.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
Static pod: kube-scheduler-k8sm1 hash: 97bd2ffa08a518ac4f55aa68c229d9f1
Static pod: kube-scheduler-k8sm1 hash: 1572155ff83eb3602b8e421ee12fa772
[apiclient] Found 1 Pods for label selector component=kube-scheduler
[upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
[upload-config] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.14" in namespace kube-system with the configuration for the kubelets in the cluster
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.14" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.14.1". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.

ubuntu@k8sm1:~$ kubectl version --short
Client Version: v1.13.8
Server Version: v1.14.1

```

- Upgrade kubectl

```
ubuntu@k8sm1:~$ sudo apt-mark unhold kubectl
Canceled hold on kubectl.
ubuntu@k8sm1:~$ 
ubuntu@k8sm1:~$ sudo apt install -y kubectl=1.14.1-00
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following package was automatically installed and is no longer required:
  libdumbnet1
Use 'sudo apt autoremove' to remove it.
The following packages will be upgraded:
  kubectl
1 upgraded, 0 newly installed, 0 to remove and 4 not upgraded.
Need to get 8806 kB of archives.
After this operation, 3876 kB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubectl amd64 1.14.1-00 [8806 kB]
Fetched 8806 kB in 2s (5038 kB/s)  
(Reading database ... 67344 files and directories currently installed.)
Preparing to unpack .../kubectl_1.14.1-00_amd64.deb ...
Unpacking kubectl (1.14.1-00) over (1.13.8-00) ...
Setting up kubectl (1.14.1-00) ...
ubuntu@k8sm1:~$ kubectl version --short
Client Version: v1.14.1
Server Version: v1.14.1
ubuntu@k8sm1:~$ sudo apt-mark hold kubectl
kubectl set on hold.
```

- Upgrade kubelet

```
ubuntu@k8sm1:~$ sudo apt-mark unhold kubelet                                        
Canceled hold on kubelet.
ubuntu@k8sm1:~$ sudo apt install -y kubelet=1.14.1-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libdumbnet1
Use 'sudo apt autoremove' to remove it.
The following packages will be upgraded:
  kubelet
1 upgraded, 0 newly installed, 0 to remove and 4 not upgraded.
Need to get 21.5 MB of archives.
After this operation, 14.8 MB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.14.1-00 [21.5 MB]
Fetched 21.5 MB in 3s (7855 kB/s)
(Reading database ... 67344 files and directories currently installed.)
Preparing to unpack .../kubelet_1.14.1-00_amd64.deb ...
Unpacking kubelet (1.14.1-00) over (1.13.8-00) ...
Setting up kubelet (1.14.1-00) ...
ubuntu@k8sm1:~$ sudo apt-mark hold kubelet
kubelet set on hold.
ubuntu@k8sm1:~$ kubectl get nodes
NAME    STATUS   ROLES    AGE     VERSION
k8sm1   Ready    master   3h21m   v1.14.1
k8sm2   Ready    <none>   3h19m   v1.13.8
```

- Upgrade worker node

No need to upgrade the controller conponents.
Only need to upgrade kubelet, kubeadm and kubectl
```
sudo apt-mark unhold kubelet kubeadm kubectl
sudo apt install -y kubeadm=1.14.1-00
sudo apt install -y kubectl=1.14.1-00 
sudo apt install -y kubelet=1.14.1-00 
sudo apt-mark hold kubeadm kubectl kubelet
```

After upgraded the kubelet, we can confirm the kubelet version from master
node.
```
ubuntu@k8sm1:~$ kubectl get nodes
NAME    STATUS   ROLES    AGE     VERSION
k8sm1   Ready    master   3h26m   v1.14.1
k8sm2   Ready    <none>   3h23m   v1.14.1
```

- refer

[kubeadm Upgrade 1.14](https://v1-15.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade-1-14/)

