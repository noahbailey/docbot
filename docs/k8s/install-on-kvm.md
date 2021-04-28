# Installing Kubernetes on KVM

## Prepare the nodes

My setup will be four VMs: 

* `kube-admin` - Control plane/master node
* `kube-0` - Compute node #1
* `kube-1` - Compute node #2
* `kube-2` - compute node #3

I can generate them using my kvmgr tool: 

    kvmgr.sh -c 2 -m 2048 -d 10G -o focal -n kube-admin

    kvmgr.sh -c 2 -m 2048 -d 20G -o focal -n kube-0
    kvmgr.sh -c 2 -m 2048 -d 20G -o focal -n kube-1
    kvmgr.sh -c 2 -m 2048 -d 20G -o focal -n kube-2

After the VMs are up, add the IPs to `/etc/hosts`

    192.168.122.42  kube-admin
    192.168.122.89  kube-0
    192.168.122.11  kube-1
    192.168.122.102 kube-2

### Kernel config

This procedure is run on all four nodes before installing kubernetes. 

Configure the bridge module to load on every boot: 

    sudo modprobe br_netfilter

Configure the network stack to bridge correctly: 

```sh
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

### Install Containerd

Containerd is the docker runtime for this setup. 

    sudo apt install -y containerd

### Install Kubelet

First, install the prereqs for HTTPS repositories: 

    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl

Import the k8s keyring: 

    sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

Add the Apt repo: 

    echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-focal main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

Install the packages on the server: 

    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl

Optionally, pin the current package versions to prevent upgrades. 

    sudo apt-mark hold kubelet kubeadm kubectl

### Install admin node tools


Create a kubeadm-config.yaml file: 

```yaml
# kubeadm-config.yaml
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta2
kubernetesVersion: v1.21.0
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
```


Begin the init process: 

    sudo kubeadm init

This take a few minutes to start the kubelet. 

When the command completes successfully: 

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

Check the cluster status: 

```
$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2m23s
```

## Adding a node to the cluster

This command needs to be run on the rest of the nodes, `kube-0`, `kube-1`, `kube-2`. 

    kubeadm join 192.168.122.42:6443 --token d3elti.x8dheeghld76txud \
        --discovery-token-ca-cert-hash sha256:4dc30c96de01f73c26ea645fbf34e45822c232c04fb1df58aca3a58df7c9b827 

### Install Calico

Calico provides the pod network abstraction layer for this cluster.

Install it from the manifest: 

    kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

Check status: 

    kubectl get pods -n kube-system

At this point the system should be ready: 

```
$ kubectl get nodes
NAME         STATUS   ROLES                  AGE     VERSION
kube-0       Ready    <none>                 8m19s   v1.21.0
kube-1       Ready    <none>                 7m35s   v1.21.0
kube-2       Ready    <none>                 7m2s    v1.21.0
kube-admin   Ready    control-plane,master   32m     v1.21.0

```

The cluster is now complete, though barebones. Additional considerations should be made for shared storage and load balancing. 