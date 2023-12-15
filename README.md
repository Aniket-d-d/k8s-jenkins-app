# k8s-jenkins-app

# Kubernetes Cluster and Jenkins Integration

This repository explores a realtime DevOps project scenario where one can deploy a 2 tier K8s application on one server from onother server using CI/CD.
This repository provides instructions and configurations for setting up a Kubernetes cluster on an EC2 instance named "dev-server" using Minikube. Additionally, it includes guidance on setting up a separate instance named "ci-server" with Jenkins to deploy Kubernetes applications on the "dev-server" Minikube cluster.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Setting up "dev-server" with Minikube](#setting-up-dev-server-with-minikube)
- [Setting up "ci-server" with Jenkins](#setting-up-ci-server-with-jenkins)
- [Connecting "dev-server" as Jenkins Agent Node](#connecting-dev-server-as-jenkins-agent-node)
- [Deploying Kubernetes Application with Jenkins](#deploying-kubernetes-application-with-jenkins)
- [Accessing the Application](#accessing-the-application)
- [Adding "dev-server" as Known host](#adding-dev-server-as-known-host)
- [References](#refernces)

  
## Prerequisites

Make sure you have the following prerequisites before starting the setup:

- AWS account with EC2 instances created (e.g., "dev-server" and "ci-server").
  "dev-server" should have atleast 2 vcpus and atleast 20 Gb of storage. 
- SSH access to both instances.
- Basic knowledge of Kubernetes and Jenkins.
- DockerHub account.

## Setting up "dev-server" with Minikube

1. Connect to "dev-server" using SSH.
2. Enter: ``` sudo apt-get update ``` to update.
3. Install docker following the [official documentation](https://docs.docker.com/engine/install/ubuntu/)
   Or Simply ``` sudo apt install docker.io ``` do the work.
4. Give default user ```ubuntu``` access to docker group.
   ```sudo usermod -aG docker ubuntu``` or ``sudo usermod -aG docker $USER```
5. Install Minikube following the [official documentation](https://minikube.sigs.k8s.io/docs/start/).
   Comand to install Minikube:
   ```
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
   sudo install minikube-linux-amd64 /usr/local/bin/minikube

   ```
6. Install kubectl following the [official documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).
   Commans to Install kubectl:
   ```
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
   echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   kubectl version --client
   ```
8. Install Java from the Jenkins official documentation [page](https://www.jenkins.io/doc/book/installing/linux/).
   This Java installation is required so that "ci-server" Jenkins can connect to "dev-server" as agent in future.
9. Start Minikube with command: ```minikube start```
10. You are done with initial setup for "dev-server".
11. You can access cluster using ```kubectl get all```.


## Setting up "ci-server" with Jenkins

1. Connect to "ci-server" using SSH.
2. Enter: ``` sudo apt-get update ``` to update.
3. Install jenkins following the [official documentation](https://www.jenkins.io/doc/book/installing/linux/).
   Commands to Install Jenkins and Java:
   ```
   sudo apt update
   sudo apt install fontconfig openjdk-17-jre
   java -version
   ```
   ```
   sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
     https://pkg.jenkins.io/debian/jenkins.io-2023.key
   echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
     https://pkg.jenkins.io/debian binary/ | sudo tee \
     /etc/apt/sources.list.d/jenkins.list > /dev/null
   sudo apt-get update
   sudo apt-get install jenkins
   ```
4. Access the Jenkins on "ci-server" port 8080.
   Complete the Initial setup process for Jenkins.
   Initial Password Command: ```sudo cat /var/lib/jenkins/secrets/initialAdminPassword```


## Connecting "dev-server" as Jenkins Agent Node

While working with Jenkins it is advised to build jobs in a distributed builds architecture use nodes, agents, and executors, which are distinct from the Jenkins controller itself. Read more [here](https://www.jenkins.io/doc/book/managing/nodes/).

Before creating/adding a node/agent on jenkins, we need to create private SSH key so that we can connect "ic-server" with "dev-server".

Steps to setup private SSH key:
1. Connect to "ci-server" using SSH.
2. If you use ```ls -la``` at your home location, you might see ```.ssh``` directory.
3. Enter Command: ```cd .ssh```.
4. Enter Command: ```ssh-keygen``` and press enter 4 times.
   Private Key: id_rsa and Public Key: id_rsa.pub will be created.
5. Copy the Contents of id_rsa.pub key.
6. Connect to "dev-server" using SSH.
7. If you use ```ls -la``` at your home location, you might see ```.ssh``` directory.
8. Enter Command: ```cd .ssh```.
9. If you use ```ls```, you will see a file named 'authorized_keys'.
10. Paste the Contents of id_rsa.pub key to the botton of 'authorized_keys' file. **Note: Do not remove older keys from "authorized_keys" file.**
11. Done with SSH key part.

Steps to create Node-Agent on Jenkins:
1. Access the Jenkins on "ci-server" port 8080.
2. Click ```Manage Jenkins```
3. Click ```Nodes``` option.
4. Click on ```+ New Node``` button.
5. Enter Node Name of your choice. I preferred ```dev-server``` same as agent instance name. Select ```Type``` as Permanent as this is the only external node. Click ```Create```.
6. Enter Description. Set ```Number of executors``` as ```1```. Generally it is set as per the availability of cps/vcpus on agent node.
7. Enter ```Remote root directory``` where you would like to set Jenkins workplace on "dev-server". In my case it is : ```/home/ubuntu/jenkins-agent-workspace```.
8. Enter```labels``` as ```dev-server```. Set Usage to : "Only build Jobs with label expression matching this node".
9. Set Launch Method as ```Launch agents via SSH```.
10. In Host enter "dev-server" IP address.
11. Create Credentials with Kind: ```SSH Username with private key```.
    Set ```ubuntu``` username and id : ```agent-connect-key```. And put the contents of id_rsa file in ```Private Key``` Enter directly box.
12. Set ```Host Key Verification Strategy``` as ```Non-Verifying Verification Strategy.```
    In my case it ```Known hosts file verification strategy```. To do that see below.
14. Save and launch the agent. You can build jobs on "dev-server" now.


 ## Deploying Kubernetes Application with Jenkins

Build a new Job on jenkins and copy the Jenkinsfile from this repo to pipeline script. Jenkins will deploy the application on minikube cluster in "dev-server".


## Accessing the Application
1. Connect to "dev-server" using SSH.
2. Enter command: kubectl port-forward --address 0.0.0.0 service/flask-app-service 30009:5001
   Though port-forwarding is not advised, because of minikube I am using it for demonstartion purpose.
3. Access the port 30009 on "dev-server" instance. You can Access the Application over there. (e.g http://dev-server-ip:30009)
   or you also access the application internally inside dev-server using ```minikube service flask-app-service```. You will be given internal IP, use curl on it. 

## Adding "dev-server" as Known host

To add "dev-server" as Known hosts for Jenkins's ```Known hosts file verification strategy```.
1. Connect to "ci-server" using SSH.
2. Enter command: ```mkdir -p /var/lib/jenkins/.ssh```
3. Enter command: ```sudo su -```. To add "dev-server" as known host we need super user access.
4. Enter command: ```ssh-keyscan -H dev-server-IP >> /var/lib/jenkins/.ssh/known_hosts``` This will tell jenkins that "dev-server" is a known and authenticated host.
5. Done. You can connect "dev-server" as agent-node.


## References
- Jenkins documentation: [https://www.jenkins.io/doc/](https://www.jenkins.io/doc/).
- Minikube documentation: [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/).
- Pod to Pod Communication: [https://kubernetes.io/docs/tutorials/services/connect-applications-service/](https://kubernetes.io/docs/tutorials/services/connect-applications-service/).
- Flask application Repo: [ak-flask-docker](https://github.com/Aniket-d-d/ak-flask-docker).

**Note: Make sure you change the dockerhub username in flask-app-pod.yaml file on line no 10.

Thankyou.
