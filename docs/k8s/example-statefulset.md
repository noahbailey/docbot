# Example StatefulSet Manifest

StatefulSet's are meant to be used for applications that hold state, like a database or disk cache. 

The key difference is that a statefulset is created with a deterministic name, and has the ability to attach persistent storage directly. 

File: `thingybase-deployment.yaml`

```yaml
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: thingybase
  labels:
    app: thingybase
spec:
  serviceName: thingyservice
  replicas: 1
  selector:
    matchLabels:
      app: thingybase
  template:
    metadata:
      labels:
        app: thingybase
    spec:
      terminationGracePeriodSeconds: 300
      containers:
      - name: thingybase
        image: ongo/thingybase:latest
        ports:
        - containerPort: 9000
          name: http
        volumeMounts:
        - name: data
          mountPath: /var/lib/thingybase
        env: 
          - name: JAVA_OPTS
            value: -Xms256m -Xmx512m
          - name: FOO
            value: "bar"
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes:
        - ReadWriteOnce
      storageClassName: ssd
      resources:
        requests:
          storage: 10Gi

```

Apply the manifest to your cluster: 

    kubectl apply -f thingybase-statefulset.yaml

## Statefulset Service

Statefulsets are also used with services. The deterministic name of the pod created will be `<podname>-<#>.<servicename>`, for example `thingybase-0.thingyservice` when it is accessed by other pods. 

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: thingyservice
  labels:
    app: thingybase
spec:
  ports:
  - port: 9000
    name: http
  selector:
    app: thingybase
```
