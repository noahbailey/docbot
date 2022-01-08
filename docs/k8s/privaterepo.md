# Private Repos

For example, pulling private images from GitHub Container Repo. 

## Docker login

First, login to docker: 

```
$ docker login ghcr.io
Username: ongo 
Password: ********

Login Succeeded
```

## Encode credentials

Create a base64 encoded string from the docker credentials: 

    base64 -w 0 ~/.docker/config.json


## Create Kubernetes secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ghcrio
data:
  .dockerconfigjson: AAAAA==
type: kubernetes.io/dockerconfigjson
```

View the submitted data: 

    kubectl get secret ghcrio --output=yaml

## Pull a container from a private repo

The credentials can be specified using `imagePullSecrets` in the deployment manifest: 

```yaml
apiVersion: v1
kind: Deployment
metadata:
  name: foobar
spec:
  containers:
  - name: foobar
    image: ghcr.io/ongo/foobar/foobar:5.6.7a
  imagePullSecrets:
  - name: ghcrio
```
