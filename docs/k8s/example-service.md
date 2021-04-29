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

## Ingress Service

File: `donixer-ingress.yaml`

```yml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: donixer-ingress
spec:
  rules:
  - host: donixer.cool
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: donixer-service
            port:
              number: 8000
```

Apply the manifest to your cluster: 

    kubectl apply -f donixer-ingress.yaml