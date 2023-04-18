# Setup Jenkins on K8s
This is an example Repo containing the necessary Kubernetes-Manifests to get started with Jenkins on K8s. 
The manifests for the setup are located in the ```k8s-jenkins-setup```-folder.   

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

# Use Jenkins
You define your Pipeline in a File called **Jenkinsfile**.  
A very basic one (doesn't actually do anything xd) would look like this:
```
pipeline {
    agent any
    stages {
        stage('test') {
            steps {
                sh 'echo "Im testing this."'
            }
        }
        stage('build') {
            steps {
                sh 'echo "Im building..."'
            }
        }
        stage('deploy') {
            steps {
                sh 'echo "Deploying..."'
            }
        }
    }
}
```

***To do the specified steps in the actual Jenkinsfile you have to install the required Plugins on the Jenkins-Server.***  
Also you have to specify credentials e.g. by using the **Jenkins Credentials Plugin**.  
```Dashboard > Manage Jenkins > Credentials > System > Global Credentials```

## Create the pipeline on the Jenkins server
Create a new multibranch pipeline and select the Repository URL as well as the Jenkinsfile location.  
Take care of the notation of the manifests. I prefixed them to ensure they are created in the right order.  

YOU NEED TO INSTALL THE KUBERNETES PLUGIN AND SET UP A NEW CLOUD TO USE IT!!!!  
```Manage Jenkins > install plugins > kubernetes, git, pipeline etc```  
```Manage Jenkins > Manage nodes > Configure Clouds > Kubernetes```  
Also configure the Jenkins URL and Jenkins tunnel to where the Jenkins Server runs and where the ClusterIP JLNP service runs.  

Kubernetes removed Docker so you cant mount docker.sock anymore. Use Kaniko for building and pushing images..
```kubectl create secret docker-registry dockercred --docker-server=https://index.docker.io/v1/ --docker-username
=SECRET --docker-password=SECRET --docker-email=SECRET -n devops-tools```