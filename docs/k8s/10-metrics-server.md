# Kubernetes Metrics Server

By default, there are no metrics at all in K8s. 

To get basic data on resource usage, you must install `metrics-server`. 

## Setup

Download the manifest: 

    wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

Edit the manifest to add the TLS flag to the args on approx line 136: 

```yaml
...
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --kubelet-insecure-tls                                            # ADD THIS LINE
        image: k8s.gcr.io/metrics-server/metrics-server:v0.4.4
...
```

Apply the manifest: 

    kubectl apply -f ./components.yml


## Per-Pod metrics:

```
$ kubectl top pod elasticsearch-0
NAME              CPU(cores)   MEMORY(bytes)   
elasticsearch-0   10m          855Mi 
```

## Per-Node metrics: 

```
$ kubectl top node kube-0
NAME     CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
kube-0   323m         16%    3241Mi          84% 
```
