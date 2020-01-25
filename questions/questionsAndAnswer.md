
1. Extract log lines corresponding to error `file-not-found` and write them to
   `/tmp/foobar`

```
$ kubectl logs foobar | grep "file-not-found" > /tmp/foobar
```

2. List all PVs sorted by name and save the kubectl output to `/tmp/my_volumes`.
   Use `kubectl`'s own functionally for sorting the output.

```
$ kubectl get pv --sort-by=.metadata.name > /tmp/my_volumes
```

3. Ensure a single instance of Pod `nginx` is running on each node of the
   kubernetes cluster where nginx also represents the image name whcich has to
   be used. Don't override any taints currently in place.

   Use Daemonsets to complete this task and use `ds.test` as Daemonset name

```
# use deployment for a template of daemonSet
$ kubectl run nginx --image=nginx --restart=Always --dry-run -o yaml > ds.yaml

ubuntu@k8smaster:~/ds$ cat ds.yaml 
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds.test
spec:
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
```


4. Add an init container to nginx pod. The init container should create an
   empty file named `/tmp/ready`.
   If `/tmp/ready` is not exist, the Pod should exit.
   Once he spec file has been updated with the init container definition, the
   Pod should be created

```
ubuntu@k8smaster:~$ cat initpod.yaml        
apiVersion: v1                              
kind: Pod                                   
metadata:                                   
  labels:                                   
    run: nginx                              
  name: nginx                               
spec:                                       
  containers:                               
  - image: nginx                            
    name: nginx                             
    livenessProbe:                          
      exec:                                 
        command: ["test","-e","/tmp/ready"] 
    volumeMounts:                           
    - name: workdir                         
      mountPath: "/tmp"                     
  initContainers:                           
  - name: touch                             
    image: busybox                          
    command: ["/bin/sh"]                    
    args: ["-c", "touch /tmp/ready"]        
    volumeMounts:                           
    - name: workdir                         
      mountPath: "/tmp"                     
  volumes:                                  
  - name: workdir                           
    emptyDir: {}                            
```

5. Create a pod named `nrmc` with a single container for each of the following
   images running inside(there may be between 1 and 4 images specified): 
   `nginx` + `redis` + `memcached` + `consul`

```
ubuntu@k8smaster:~$ cat 4images.yaml  
apiVersion: v1                        
kind: Pod                             
metadata:                             
  labels:                             
    run: nrmc                         
  name: nrmc                          
spec:                                 
  containers:                         
  - image: nginx                      
    name: nginx                       
  - image: redis                      
    name: redis                       
  - image: memcached                  
    name: memcached                   
  - image: consul                     
    name: consul                      
  dnsPolicy: ClusterFirst             
  restartPolicy: Never                
```

6. Schedule a Pod as follows:
   Name: `nginx-pod`
   Image: `nginx`
   Node selector: `zone=zone0`

```
ubuntu@k8smaster:~$ kubectl get node -L zone
NAME        STATUS   ROLES    AGE   VERSION   ZONE
k8smaster   Ready    master   31d   v1.17.0   zone0
k8snode1    Ready    <none>   17d   v1.17.0
k8snode2    Ready    <none>   19d   v1.17.0

ubuntu@k8smaster:~$ cat nodeSelector.yaml    
apiVersion: v1                               
kind: Pod                                    
metadata:                                    
  labels:                                    
    run: nginx                               
  name: nginxzone0                           
spec:                                        
  containers:                                
  - image: nginx                             
    name: nginx                              
  dnsPolicy: ClusterFirst                    
  restartPolicy: Never                       
  nodeSelector:                              
    zone: zone0                              

ubuntu@k8smaster:~$ kubectl get pod -o wide
NAME         READY   STATUS    RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
nginxzone0   1/1     Running   0          15s     10.244.0.11   k8smaster   <none>           <none>

7. Create a deployment as follows
   Name: `nginx-app`
   Using container `nginx` with version `1.10.2-alpin`
   Replicas: `3`
   Next, deploy the app with new version `1.13.0-alpine` by performing a rolling 
   update and record that update.
   Finally, rollback that update to the previous version `1.10.2-alpine`



8. Create and configure the service `front-end-service` so it's accessible
   through NodePort and routes to the existing pod named `front-end`

9. Create a Pod as follows:
   Name: jenkins
   Using image: jenkins
   Namespace: website

10. Create a deployment spec file that will launch 7 replicas of `redis` image
    with the label: `app=dev`
    The name of the deployment should be `lucky7`

11. Create a file `/tmp/foo.txt` that lists all pods that implement Service
    `foo` in Namespace `production`

12. Create a secret as follows:
    Name: `top-secret`
    Credential: `alice or username:bob`

    Create a Pod named `pod-secrets-via-file` using the `redis` image which mounts a
    secret named `top-secret` at `/secrets`
    Create a second Pod named `pod-secrets-via-env` using the `redis` image
    which exports credential as `TOPSECRET`

13. Create a pod as follows:
    Name: `non-persistent-redis`
    Container image: `redis`
    Named-volume with name: `cache-control`
    Mount path: `/data/redis`
    It should launch in the `pre-prod` namespace and the volume MUST NOT be 
    persistent.

14. Scale a deployment `webserver` to `6` pods.

15. Check to see how many nodes are `ready` (not including nodes tainted 
    NoSchedule) and write the number to `/tmp/nodenum`

16. From the Pod label `name=cpu-utilizer`, find pods running high CPU 
    workloads and write the name of the Pod consuming most CPU to the 
    file /opt/cpu.txt (which already exists)

17. Create a deployment as follows
    Name: `nginx-dns`
    Exposed via a service: `nginx-dns`
    Ensure that the service & pod are accessible via their respective DNS 
    records
    The container(s) within any Pod(s) running as a part of this deployment 
    should use the `nginx` image

    Next, use the utility `nslookup` to look up the DNS records of the service 
    & pod and write the output to `/opt/service.dns` and `/opt/pod.dns`
     respectively.

    Ensure you use the `busybox:1.28` image(or earlier) for any testing, 
    The latest release has an unpstream bug which impacts thd use of nslookup.

18. Create a snapshot of the etcd instance running at `https://127.0.0.1:2379`
    saving the snapshot to the file path `/backup/etcd-snapshot.db`

    The etcd instance is running etcd version `3.1.10`
    The following TLS certificates/key are supplied for connecting to the 
    server with etcdctl

      CA certificate: `/opt/KUCM00302/ca.crt`
      Client certificate: `/opt/KUCM00302/etcd-client.crt`
      Clientkey: `/opt/KUCM00302/etcd-client.key`

19. Set the node labelled with `name=ek8s-node-1` as `unavailable` and 
    reschedule all the pods running on it.

20. A Kubernetes worker node, labelled with `name=wk8s-node-0` is in state 
    `NotReady` . Investigate why this is the case, and perform any appropriate 
    steps to bring the node to a Ready state, ensuring that any changes are 
    made permanent.

    Hints:
    
    You can ssh to the failed node using $ ssh wk8s-node-0
    You can assume elevated privileges on the node with the following command 
    `$ sudo -i`

21. Configure the `kubelet` systemd managed service, on the node labelled with 
    `name=wk8s-node-1`, to launch a Pod containing a single container of image 
    `nginx` named `myservice` automatically. 
    Any spec files required should be placed in the `/etc/kubernetes/manifests`
    directory on the node.

    Hints:
    
    You can ssh to the failed node using $ ssh wk8s-node-1
    You can assume elevated privileges on the node with the following command 
    `$ sudo -i` 

22. In this task, you will configure a new Node, `ik8s-node-0`, to join a 
    Kubernetes cluster as follows:

    - Configure kubelet for automatic certificate rotation and ensure that both
    server and client CSRs are automatically approved and signed as appropnate 
    via the use of RBAC.
    - Ensure that the appropriate cluster-info `ConfigMap` is created and 
    configured appropriately in the correct namespace so that future Nodes can 
    easily join the cluster
    - Your bootstrap kubeconfig should be created on the new Node at 
    `/etc/kubernetes/bootstrap-kubelet.conf` (do not remove this file once your 
    Node has successfully joined the cluster)
    - The appropriate cluster-wide CA certificate is located on the Node at 
    `/etc/kubernetes/pki/ca.crt`. You should ensure that any automatically 
    issued certificates are installed to the node at `/var/lib/kubelet/pki`
    and that the `kubeconfig` file for `kubelet` will be rendered at 
    `/etc/kubernetes/kubelet.conf` upon successful bootstrapping
    - Use an additional group for bootstrapping Nodes attempting to join the 
    cluster which should be called
    `system:bootstrappers:cka:default-node-token`
    - Solution should start automatically on boot, with the systemd service 
    unit file for kubelet available at `/etc/systemd/system/kubelet.service`
    To test your solution, create the appropriate resources from the spec 
    file located at /opt/..../kube-flannel.yaml This will create the necessary 
    supporting resources as well as the `kube-flannel` -ds `DaemonSet` . 
    You should ensure that this DaemonSet is correctly deployed to the single 
    node in the cluster.

    Hints:
    
    - kubelet is not configured or running on `ik8s-master-0` for this task, and you should not attempt to configure it.
    - You will make use of TLS bootstrapping to complete this task.
    - You can obtain the IP address of the Kubernetes API server via the following command `$ ssh ik8s-node-0 getent hosts ik8s-master-0`
    - The API server is listening on the usual port, 6443/tcp, and will only server TLS requests
    - The kubelet binary is already installed on ik8s-node-0 at /usr/bin/kubelet . You will not need to deploy kube-proxy to the cluster during this task.
    - You can ssh to the new worker node using `$ ssh ik8s-node-0`
    - You can ssh to the master node with the following command `$ ssh ik8s-master-0`
    - No further configuration of control plane services running on `ik8s-master-0` is required
    - You can assume elevated privileges on both nodes with the following command `$ sudo -i`
    - Docker is already installed and running on `ik8s-node-0`

23. Given a partially-functioning Kubenetes cluster, identify symptoms of 
    failure on the cluster. Determine the node, the failing service and take 
    actions to bring up the failed service and restore the health of the 
    cluster. Ensure that any changes are made permanently.

    The worker node in this cluster is labelled with `name=bk8s-node-0` 
    
    Hints:

    - You can ssh to the relevant nodes using `$ ssh $(NODE)` where `$(NODE)` is one of `bk8s-master-0` or `bk8s-node-0`
    - You can assume elevated privileges on any node in the cluster with the following command `$ sudo -i`

24. Creae a persistent volume with name `app-config` of capacity `1Gi` and 
    access mode `ReadWriteOnce`. The type of volume is `hostPath` and its 
    location is `/srv/app-config`

```
ubuntu@k8smaster:~/pv$ cat pv.yaml                                                                     
apiVersion: v1                                                                                         
kind: PersistentVolume                                                                                 
metadata:                                                                                              
  name: app-config                                                                                     
spec:                                                                                                  
  storageClassName: ""                                                                                 
  capacity:                                                                                            
    storage: 1Gi                                                                                       
  accessModes:                                                                                         
    - ReadWriteOnce                                                                                    
  hostPath:                                                                                            
    path: "/srv/app-config"                                                                            
ubuntu@k8smaster:~/pv$ kubectl create -f pv.yaml                                                       
persistentvolume/app-config created                                                                    
ubuntu@k8smaster:~/pv$ kubectl get pv                                                                  
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
app-config   1Gi        RWO            Retain           Available                                   10s

```

