# MetalLB Load balancer

MetalLB is used on a cluster that does not have access to cloud environments, enabling the use of `type=LoadBalancer` services. 

References: 

* https://metallb.universe.tf/installation/
* https://metallb.universe.tf/concepts/layer2/
* https://metallb.universe.tf/configuration/#layer-2-configuration

## Configure Kube-Proxy

Reconfigure the kube-proxy to allow L2 networking reconfiguration

```
kubectl get configmap kube-proxy -n kube-system -o yaml | \
  sed -e "s/strictARP: false/strictARP: true/" | \
  kubectl diff -f - -n kube-system

kubectl get configmap kube-proxy -n kube-system -o yaml | \
  sed -e "s/strictARP: false/strictARP: true/" | \
  kubectl apply -f - -n kube-system
```
## Install MetalLB

Apply the manifests to install the service: 

    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.6/manifests/namespace.yaml
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.6/manifests/metallb.yaml
    kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"

Check the installation

```
$ kubectl get all -n metallb-system

NAME                              READY   STATUS    RESTARTS   AGE
pod/controller-64f86798cc-dsb8t   1/1     Running   0          104s
pod/speaker-22wwl                 1/1     Running   0          104s
pod/speaker-6lxrs                 1/1     Running   0          104s
pod/speaker-nnpkb                 1/1     Running   0          104s
pod/speaker-rbvpn                 1/1     Running   0          104s

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/speaker   4         4         4       4            4           kubernetes.io/os=linux   104s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           104s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-64f86798cc   1         1         1       104s
```

## Configure MetalLB

Configuration in L2 mode is fairly simple, only needing to input an ipv4 range: 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.122.240-192.168.122.250
```

## Using MetalLB

Metallb works like any other loadbalancer provider once configured. For example: 

```
$ kubectl get service
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)          AGE
guestbook      LoadBalancer   10.107.63.180    192.168.122.240   3000:30829/TCP   4s
```

Visiting `http://192.168.122.240:3000` will expose the `guestbook` service, very similarly to how a cloud load balancer would work. 

It can also be used in conjunction with [Nginx Ingress](/k8s/nginx-ingress).
