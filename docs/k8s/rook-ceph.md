# Rook - Ceph on Kubernetes

References: 

* https://rook.io/docs/rook/v1.6/ceph-quickstart.html

## Preparing the cluster

Each node that will have an OSD must have at least one additional block device with no partition tables or LVM pvs. 

For VMs, simply create an additional virtio block device and reboot. 

The disk layout should look something like this: 

```
$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda     252:0    0   20G  0 disk 
├─vda1  252:1    0 19.9G  0 part /
├─vda14 252:14   0    4M  0 part 
└─vda15 252:15   0  106M  0 part /boot/efi
vdb     252:16   0   20G  0 disk                  <---- The OSD will live here
```

In my lab, there are three compute/storage nodes, and one master node: 

Node Name | Roles | Storage
--------- | ----- | -------
kube-admin | master, control plane | 10G (root)
kube-0 | worker, ceph | 10G (root), 20G (OSD)
kube-1 | worker, ceph | 10G (root), 20G (OSD)
kube-2 | worker, ceph | 10G (root), 20G (OSD)

## Install Rook

Clone the repo and find the examples directory. 

    git clone --single-branch --branch v1.6.1 https://github.com/rook/rook.git
    cd rook/cluster/examples/kubernetes/ceph

Create the base resource set. 

    kubectl create -f crds.yaml -f common.yaml -f operator.yaml

Check that the pod is running before proceeding: 

```
$ kubectl -n rook-ceph get pod -w
NAME                                 READY   STATUS              RESTARTS   AGE
rook-ceph-operator-95f44b96c-6czkh   0/1     ContainerCreating   0          25s
rook-ceph-operator-95f44b96c-6czkh   1/1     Running             0          29s
```


## Create the cluster

Apply the cluster manifest and prepare the cluster (MONs, OSDs, etc). 

    kubectl create -f ./cluster.yaml

This will take approx 10 minutes to complete. Be patient while it starts up. 

    kubectl get pods -n rook-ceph -o wide -w

You can tell if it has successfully installed if you see the output of `lsblk` change. Note the `ceph_bluestore` filesystem that now exists on `/dev/vdb`. 

```
$ lsblk -f
...
vda
├─vda1  ext4           cloudimg-rootfs 1943530c-1f82-498b-8b82-d1b474ba22c1   12.4G    35% /
├─vda14
└─vda15 vfat           UEFI            53A4-477C                              95.2M     9% /boot/efi
vdb     ceph_bluestore
```

When complete, the pods should look a little like this: 

```
$ kubectl -n rook-ceph get pod 
NAME                                               READY   STATUS      RESTARTS   AGE
csi-cephfsplugin-m6lpq                             3/3     Running     0          5m7s
csi-cephfsplugin-ngc8p                             3/3     Running     0          5m7s
csi-cephfsplugin-provisioner-58d557d5-fr724        6/6     Running     0          5m7s
csi-cephfsplugin-provisioner-58d557d5-g7r6g        6/6     Running     0          5m7s
csi-cephfsplugin-sv2ch                             3/3     Running     0          5m7s
csi-rbdplugin-4kfsq                                3/3     Running     0          5m8s
csi-rbdplugin-7f9f9                                3/3     Running     0          5m8s
csi-rbdplugin-ft7s5                                3/3     Running     0          5m8s
csi-rbdplugin-provisioner-7bcb95bc5d-gmpp6         6/6     Running     0          5m8s
csi-rbdplugin-provisioner-7bcb95bc5d-txcxg         6/6     Running     0          5m8s
rook-ceph-crashcollector-kube-0-74fc9db597-v67zh   1/1     Running     0          2m31s
rook-ceph-crashcollector-kube-1-74bf796976-zq4rk   1/1     Running     0          2m57s
rook-ceph-crashcollector-kube-2-6fbb896bf5-s9b7t   1/1     Running     0          3m14s
rook-ceph-mgr-a-848c65cc59-whh2w                   1/1     Running     0          2m46s
rook-ceph-mon-a-758bb5d598-tlc2m                   1/1     Running     0          4m25s
rook-ceph-mon-b-c446bb45d-dm8ng                    1/1     Running     0          4m14s
rook-ceph-mon-c-6485cb8bfc-nt6mr                   1/1     Running     0          2m57s
rook-ceph-operator-95f44b96c-6czkh                 1/1     Running     0          24m
rook-ceph-osd-0-69558cb546-f4tks                   1/1     Running     0          2m31s
rook-ceph-osd-1-79d7fdc4d8-zm7v9                   1/1     Running     0          2m31s
rook-ceph-osd-2-854fb46d86-7bzbr                   1/1     Running     0          2m31s
rook-ceph-osd-prepare-kube-0-bjqdj                 0/1     Completed   0          2m9s
rook-ceph-osd-prepare-kube-1-lsvlw                 0/1     Completed   0          2m7s
rook-ceph-osd-prepare-kube-2-2pkdb                 0/1     Completed   0          2m5s
```

### Install ceph tools

Create a manifest file in ~/

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rook-ceph-tools
  namespace: rook-ceph
  labels:
    app: rook-ceph-tools
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rook-ceph-tools
  template:
    metadata:
      labels:
        app: rook-ceph-tools
    spec:
      dnsPolicy: ClusterFirstWithHostNet
      containers:
      - name: rook-ceph-tools
        image: rook/ceph:v1.6.1
        command: ["/tini"]
        args: ["-g", "--", "/usr/local/bin/toolbox.sh"]
        imagePullPolicy: IfNotPresent
        env:
          - name: ROOK_CEPH_USERNAME
            valueFrom:
              secretKeyRef:
                name: rook-ceph-mon
                key: ceph-username
          - name: ROOK_CEPH_SECRET
            valueFrom:
              secretKeyRef:
                name: rook-ceph-mon
                key: ceph-secret
        volumeMounts:
          - mountPath: /etc/ceph
            name: ceph-config
          - name: mon-endpoint-volume
            mountPath: /etc/rook
      volumes:
        - name: mon-endpoint-volume
          configMap:
            name: rook-ceph-mon-endpoints
            items:
            - key: data
              path: mon-endpoints
        - name: ceph-config
          emptyDir: {}
      tolerations:
        - key: "node.kubernetes.io/unreachable"
          operator: "Exists"
          effect: "NoExecute"
          tolerationSeconds: 5
```

Apply the manifest to create the deployment: 

    kubectl create -f ./ceph-toolbox.yaml

Create a bash alias to exec the container in `~/.bashrc`

    echo 'alias ceph-tools="kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash"' >> .bashrc
    source .bashrc

Then, execute `ceph-tools` to enter the container. You may then check the status, it should be _reasonably_ healthy:

```
[root@rook-ceph-tools-57787758df-52fld /]# ceph osd status
ID  HOST     USED  AVAIL  WR OPS  WR DATA  RD OPS  RD DATA  STATE      
 0  kube-0  1026M  18.9G      0        0       0        0   exists,up  
 1  kube-1  1026M  18.9G      0        0       0        0   exists,up  
 2  kube-2  1026M  18.9G      0        0       0        0   exists,up  
```


## Set up storage

Create the StorageClass, Filesystem (NFS), and Object Storage (S3) resource types. I will use EC (Erasure Coded) pools for better efficiency. 

### StorageClass

Apply the manifest: 

    kubectl apply -f cluster/examples/kubernetes/ceph/csi/rbd/storageclass-ec.yaml

Check if the storageClass exists: 

```
$ kubectl get storageclass
NAME              PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com   Delete          Immediate           true                   16m
```


## Object (S3)

Apply the manifest: 

    kubectl apply -f cluster/examples/kubernetes/ceph/object-ec.yaml

Check the pods: 

```
$ kubectl -n rook-ceph get pod -l app=rook-ceph-rgw
NAME                                       READY   STATUS    RESTARTS   AGE
rook-ceph-rgw-my-store-a-85d646566-4lpwq   1/1     Running   0          10m
```

## Filesystem (NFS)

Apply the manifest: 

    kubectl apply -f cluster/examples/kubernetes/ceph/filesystem-ec.yaml

Check the pods:

```
$ kubectl -n rook-ceph get pod -l app=rook-ceph-mds
NAME                                       READY   STATUS    RESTARTS   AGE
rook-ceph-mds-myfs-ec-a-7dc5c98b9c-zstpb   1/1     Running   0          8m18s
rook-ceph-mds-myfs-ec-b-7578bd7f7c-x9qrl   1/1     Running   0          8m17s
```

## Finishing up

If they are applied correctly, the output of `ceph-df` will have more entries for the created pools: 

```
[root@rook-ceph-tools-57787758df-52fld /]# ceph df
--- RAW STORAGE ---
CLASS  SIZE    AVAIL   USED    RAW USED  %RAW USED
hdd    60 GiB  57 GiB  85 MiB   3.1 GiB       5.14
TOTAL  60 GiB  57 GiB  85 MiB   3.1 GiB       5.14
 
--- POOLS ---
POOL                         ID  PGS  STORED   OBJECTS  USED     %USED  MAX AVAIL
device_health_metrics         1    1      0 B        0      0 B      0     18 GiB
replicated-metadata-pool      2   32      0 B        0      0 B      0     27 GiB
ec-data-pool                  3   32      0 B        0      0 B      0     36 GiB
my-store.rgw.control          4    8      0 B        8      0 B      0     18 GiB
my-store.rgw.meta             5    8  1.7 KiB        7  1.1 MiB      0     18 GiB
my-store.rgw.log              6    8  3.5 KiB      178  6.2 MiB   0.01     18 GiB
my-store.rgw.buckets.index    7    8      0 B       11      0 B      0     18 GiB
my-store.rgw.buckets.non-ec   8    8      0 B        0      0 B      0     18 GiB
.rgw.root                     9    8  3.9 KiB       16  2.8 MiB      0     18 GiB
my-store.rgw.buckets.data    10   32      0 B        0      0 B      0     36 GiB
myfs-ec-metadata             11   32  2.2 KiB       22  1.5 MiB      0     18 GiB
myfs-ec-data0                12   32      0 B        0      0 B      0     36 GiB
```

To test the system, create a mysql/wordpress install. 

    kubectl apply -f  cluster/examples/kubernetes/mysql.yaml
    kubectl apply -f  cluster/examples/kubernetes/wordpress.yaml

A PersistentVolumeClaim will be created on the Ceph storage pools: 

```
$ kubectl get pvc
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
mysql-pv-claim   Bound    pvc-82ae2e99-8ac1-4d2d-9e0d-a9ea6f796985   20Gi       RWO            rook-ceph-block   73s
wp-pv-claim      Bound    pvc-d9ddabd7-ec55-4936-9a6e-b48ef221fdd0   20Gi       RWO            rook-ceph-block   44s
```

It will also create a service which is accessible outside the cluster: 

```
$ kubectl get service wordpress
NAME        TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)        AGE
wordpress   LoadBalancer   10.98.174.239   192.168.122.241   80:32014/TCP   85s
```
    
This should increase the usage of the OSDs: 

```
[root@rook-ceph-tools-57787758df-52fld /]# ceph osd df
ID  CLASS  WEIGHT   REWEIGHT  SIZE    RAW USE  DATA     OMAP  META   AVAIL   %USE  VAR   PGS  STATUS
 0    hdd  0.01949   1.00000  20 GiB  1.3 GiB  262 MiB   0 B  1 GiB  19 GiB  6.28  1.00  203      up
 1    hdd  0.01949   1.00000  20 GiB  1.3 GiB  262 MiB   0 B  1 GiB  19 GiB  6.28  1.00  194      up
 2    hdd  0.01949   1.00000  20 GiB  1.3 GiB  262 MiB   0 B  1 GiB  19 GiB  6.28  1.00  198      up
                       TOTAL  60 GiB  3.8 GiB  785 MiB   0 B  3 GiB  56 GiB  6.28                   
MIN/MAX VAR: 1.00/1.00  STDDEV: 0
```

The Wordpress & Mysql components can be deleted after this. 

