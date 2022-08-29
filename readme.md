# DevOps Assessment
## _intro_

hi...

this is an assessment that consist of four stages, complete each stage as you see fit, then submit it to a github repo.

## _Prerequisites_

This assessment requires running a k8s cluster (k3d), you can run the cluster on your personal machine or for a better environment you can use any cloud service (e.g, an ec2 instance from AWS).
To complete this assessment your machine need to have the following installed :

- docker
- make (CLI tool)
- WSL (in case you are running windows)

[Mayank]: I created AWS account and git account as prerequisites.
- I have created EC2-Ubuntu instance and SSH access enabled.


## I - environment preparation

This assessment uses a lightweight k8s cluster called k3d that can simulate a cluster of more than one node on your personal machine. 

∴ the first part of the asseessment is to install _k3d_.

[Mayank]: - Fllowed below steps to prepare the require environment.
1.  $ sudo su -
2.  $ apt-get update
3.  Install Docker from official website. -> https://docs.docker.com/engine/install/ubuntu/
4.  Install Git on Ubuntu server.
5.  $ apt-get install git
6.  $ git clone https://github.com/HatimAbdullah/devops-assessment.git


## II - cluster preparation 

Once k3d is installed, you can run the command `make cluster` in the root dirctory of the project, this command will create a k3d cluster of one node. once the cluster is created you can interact with it using `kubectl`.

First, in order to prepare the cluster you will need to install _helm_ (the k8s package manager). 

Afterwards, you need to install and deploy the following services to your cluster: 

- nginx ingress 
- prometheus 

[Mayank]: - Create K3D cluster
1.  $ sudo su -
2.  $ cd /home/ubuntu/devops-assessment/
3.  $ make cluster
4.  I faced error and unable to create cluster. I have modified below parameters in "Makefile"
    - "--k3s-arg '--disable=traefik@server:0'"
   - **Corrected Makefile**
    <img width="933" alt="image" src="https://user-images.githubusercontent.com/30410957/187270559-0c90b31e-8887-4edb-932f-22b149580f7b.png">

5. Makefile result
    <img width="687" alt="image" src="https://user-images.githubusercontent.com/30410957/187270710-042eab05-6173-4bce-bdcc-6d18b4af4604.png">
    
6. Install Helm for package dependency for K8s.
    <img width="821" alt="image" src="https://user-images.githubusercontent.com/30410957/187271235-a1d68421-f4f6-4ff4-8e66-fd05b62ef939.png">
    
    <img width="818" alt="image" src="https://user-images.githubusercontent.com/30410957/187280352-af8e00aa-93e7-45e5-b4b5-655d8b43aef3.png">
    
7. Install nginx ingress using Helm with below commands and preparing repo
- helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
- helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace --set controller.publishService.enabled=true
8. Install kubectl using below command
- curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
- chmod +x kubectl
- cp ~/.local/bin/kubectl /usr/bin/
9. Verify cluster using below commands.
- kubectl get all --namespace=ingress-nginx
    <img width="685" alt="image" src="https://user-images.githubusercontent.com/30410957/187281569-729b20fd-7481-47bc-8f8d-7aefc948ce38.png">
10. Install prometheus using Helm with below commands and verification
- helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
- helm install prometheus prometheus-community/prometheus -n prometheus --create-namespace
- kubectl get all --namespace=prometheus
    <img width="641" alt="image" src="https://user-images.githubusercontent.com/30410957/187281676-82f76a5a-3e04-4d66-a5fa-f07e261e9f3b.png">


## III - service deployment

- create a dockerfile for the nodejs sample service in the root dir.
- create a deployment file for the sample service with the following conditions :
    - set a soft limit and a hard limit on the service resources consumption.
    - deploy the service without root privileges.
    - limlit the deployment capabilities to only the necessary ones.
    - set the filesystem to be read only.
    - disallow privilege escalation.
    - livenessProbe and readinessProbe must be set on the deployment file (you can find the health endpoit by browsing to /sample-service/index.js).
- create a service file for the sample service.
- create an ingress file that exposes the sample service to outside of the cluster.

all the resources mentioned above should exist on the same one namespace, and the service should be running and accessible. 

[Mayank]: - service deployment using below steps
1. $ cd manifest
2. $ kubectl apply -f app-deployment.yaml
3. $ kubectl apply -f app-service.yaml
4. $ kubectl apply -f app-ingress.yaml

## IV - bonus

- deploy jenkins (CICD tool) to your cluster using helm.
- create a pipeline in jenkins that consists of two main stages : 
    - stage 1 : build the sample service docker image and pushes it to dockerhub.
    - stage 2 : deploy the sample service to the cluster with a rollout strategy.

[Mayank]: - Installed Jenkins using helm
1. $ helm repo add jenkins https://charts.jenkins.io
2. $ helm repo update
3. $ helm install jenkins-ci jenkins/jenkins
4. $ kubectl apply -f jenkins-ingress.yaml

[Mayank]: - Build and deployment
1. Clone the repo inside the build pod
2. Build the image and tag with latest build number and push the image to docker registry.
3. Getting the last deployed image in variable and replacing the old image with new image with lastest tag as build number in app-deplyment.yaml file and finally deployed the app-deployment.yaml.

 ✨ Good Luck ✨
