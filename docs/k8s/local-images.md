# Build local images

Sometimes, you need a quick and dirty way to push locally built containers to a k8s cluster. In this case, a microk8s on a nearby VM. 

### Build the image

    docker build -t ongo/mycontainer:latest ./

### Push to the local registry

    microk8s enable registry

    docker tag ongo/mycontainer:latest localhost:32000/ongo/mycontainer:latest

    docker push localhost:32000/ongo/mycontainer:latest

