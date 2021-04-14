# MicroK8s

https://microk8s.io

## Installing

    sudo snap install microk8s --classic 

    sudo microk8s enable dashboard dns registry istio 

    sudo usermod -aG microk8s $USER

## Running

### Startup & Configure

Startup the service

    microk8s start


Add to `.bashrc`

    alias kubectl="microk8s kubectl"

Check if it's running correctly: 

```
$ kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   6d23h
```

Generate the kube config file: 

    microk8s config >> ~/.kube/config
    chmod 0600 ~/.kube/config

## Using Helm

### Install 

Install using Snap (quick & dirty) 

    sudo snap install helm --classic

Add a repository: 

    helm repo add stable https://charts.helm.sh/stable

### Play with it

    helm search repo stable

    helm install stable/mysql --generate-name

    helm ls

    helm uninstall mysql-xxx

And if you really want to kill your computer: 

    helm install stable/elasticsearch --generate-name

> You need at least 10G of memory for this, don't install this chart :(
