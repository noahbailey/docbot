# Helm Package Manager

## Install a package

    helm repo add jetstack https://charts.jetstack.io

    helm install \
        cert-manager jetstack/cert-manager \
        --namespace cert-manager \
        --create-namespace \
        --version v1.6.1

## List Packages

```
helm list -A

NAME                 	NAMESPACE            	REVISION	UPDATED                                	STATUS  	CHART                       	APP VERSION
cert-manager         	cert-manager         	1       	2022-01-17 18:26:11.45415583 -0500 EST 	deployed	cert-manager-v1.6.1         	v1.6.1     
kube-prometheus-stack	kube-prometheus-stack	1       	2022-01-14 16:58:02.442833425 +0000 UTC	deployed	kube-prometheus-stack-23.3.1	0.52.1     
metrics-server       	kube-system          	1       	2022-01-14 16:56:44.476129568 +0000 UTC	deployed	metrics-server-5.10.5       	0.5.1      
```

