# Recovering from Crashes

## Evicted pods

They're basically zombies, so it's good to kill them: 

    kubectl get pod | grep Evicted | awk '{print $1}' | xargs kubectl delete pod

