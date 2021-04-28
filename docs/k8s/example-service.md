# Example Service Manifest

File: `donixer-service.yaml`

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: donixer-service
  labels:
   app: donixer
spec:
  type: LoadBalancer
  ports:
  - name: http
    protocol: TCP
    port: 8000
  selector:
    app: donixer
```

Apply the manifest to your cluster: 

    kubectl apply -f donixer-service.yaml
