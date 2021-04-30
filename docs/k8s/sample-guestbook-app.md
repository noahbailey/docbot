# Config sample - Guestbook app

## Redis master

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/examples/master/guestbook-go/redis-master-controller.json

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/examples/master/guestbook-go/redis-master-service.json

## Redis replica

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/examples/master/guestbook-go/redis-slave-controller.json

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/examples/master/guestbook-go/redis-slave-service.json

## Guestbook app

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/examples/master/guestbook-go/guestbook-controller.json

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/examples/master/guestbook-go/guestbook-service.json

## Result 

Pods: 

```
$ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
guestbook-5rdww      1/1     Running   0          11s
guestbook-85vw6      1/1     Running   0          11s
guestbook-dd8bx      1/1     Running   0          11s
redis-master-fmr6h   1/1     Running   0          50s
redis-slave-kf89t    1/1     Running   0          21s
redis-slave-mvl8m    1/1     Running   0          21s
```

Services: 
```
$ kubectl get services
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)          AGE
guestbook      LoadBalancer   10.102.7.62      192.168.122.200   3000:30721/TCP   8s
kubernetes     ClusterIP      10.96.0.1        <none>            443/TCP          15m
redis-master   ClusterIP      10.108.118.26    <none>            6379/TCP         47s
redis-slave    ClusterIP      10.103.121.153   <none>            6379/TCP         20s
```

