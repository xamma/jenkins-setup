# Setup Jenkins on K8s
This is an example Repo containing the necessary Kubernetes-Manifests to get started with Jenkins on K8s.  

## Create Namespace
First you need to create a namespace.  
In this case its named ***devops-tools*** if you want to change it, make sure to adjust all the files.  
```
kubectl create ns devops-tools
```

## Adjust Values
You need to adjust the values in **service.yaml** and **volume.yaml** based on your preferences.  
Especially, the hostname of the node and the Service-type (Loadbalancer/Ingress etc.).  

## Apply manifests
```kubectl apply -f <name of manifests>```

## Initial configuration
Open the URL e.g. ```k8s-lb-ip:8080``` and follow the instructions.  
You can find the initial password in the pod logs.  