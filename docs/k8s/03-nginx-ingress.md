# Nginx Ingress MetalLB

Prereq: Install [MetalLB](/k8s/metallb) on the cluster. 

## Install 

Install the manifest: 

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/deploy.yaml

Check the installation status: 

```
$ kubectl -n ingress-nginx get pods 
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-ll6k6        0/1     Completed   0          62s
pod/ingress-nginx-admission-patch-xwfmx         0/1     Completed   0          62s
pod/ingress-nginx-controller-75557995f8-bszs9   1/1     Running     0          62s
```

## Configure the LoadBalancer component

On its own, the 'bare metal' configuration does not include a load balancer and relies on using NodePorts. Since this cluster has MetalLB to provide load balancing, this component can be bolted on after the main manifest has been applied. Nothing about it is specific to MetalLB, it simply creates a LB service in the Nginx namespace. 

Add the manifest for the load balancer: 

```yaml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.45.0
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  type: LoadBalancer
  externalTrafficPolicy: Local
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: http
    - name: https
      port: 443
      protocol: TCP
      targetPort: https
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/component: controller
```

After configured, the load balancer will expose its IPv4 address: 

```sh
noah@kube-admin:~/ingress$ kubectl -n ingress-nginx get service
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.98.99.38     192.168.122.201   80:31347/TCP,443:30495/TCP   35m
ingress-nginx-controller-admission   ClusterIP      10.102.49.124   <none>            443/TCP                      35m

```

## Configure Ingress for a Service

After ingress had been configured, a configuration is added for an existing service.

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: foobar-ingress
spec:
  rules:
  - host: foo.bar.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: foobar-service
            port:
              number: 3000
```


