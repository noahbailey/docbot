# Example Deployment Manifest

File: `donixer-deployment.yaml`

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: donixer
  labels:
    app: donixer
spec:
  replicas: 3
  selector:
    matchLabels:
      app: donixer
  template:
    metadata:
      labels:
        app: donixer
    spec:
      containers:
      - name: donixer
        image: ongo/donixer:latest
        ports:
        - name: http
          containerPort: 8000
          protocol: TCP
        env: 
          - name: FOO
            value: "bar"
        livenessProbe:
          tcpSocket:
            port: http
          initialDelaySeconds: 20
          periodSeconds: 10
```

Apply the manifest to your cluster: 

    kubectl apply -f donixer-deployment.yaml