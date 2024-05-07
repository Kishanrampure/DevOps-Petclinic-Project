
# DevOps - Petclinic - Project
 - Project
Implementing a DevOps CI/CD pipeline for the Sample Petclinic Application in a production level environment.

Understanding the Petclinic application with a few diagrams

![Petclinic dashboard](https://github.com/Kishanrampure/DevOps-Petclinic-Project/assets/121344253/2cd46529-ef4e-4166-ad1e-952405c68e05)


## Prerequisites
- Install JKD 17
- Install Jenkins
- Install Trivy 
- Install Git
- Install Docker and Docker Compose
- Sonarqube
## Installation instructions are below on ubuntu

Install Prerequisites
```bash
  #!/bin/bash

###################################################################
#JDK installation 
###################################################################
sudo apt update
sudo apt install openjdk-17-jdk -y

###################################################################
#Docker installation 
###################################################################
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


sudo apt update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo systemctl enable docker
sudo apt install docker-compose -y


###################################################################
#Jenkins installation 
###################################################################
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt-get install jenkins -y
sudo usermod -aG root jenkins
sudo chmod 777 /var/run/docker.sock
sudo systemctl enable jenkins
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

###################################################################
#Git installation 
###################################################################
sudo apt install git-all -y

###################################################################
#Trivy installation 
###################################################################
sudo apt-get install wget apt-transport-https gnupg lsb-release -y

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y

###################################################################
#Sonarqube installation 
###################################################################
# Install Postgresql 15
sudo apt upgrade -y

sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null

sudo apt update
sudo apt-get -y install postgresql postgresql-contrib
sudo systemctl enable postgresql

sudo passwd postgres
su - postgres
###################################################################
#Create Database for Sonarqube
createuser sonar
psql 

ALTER USER sonar WITH ENCRYPTED password 'sonar';
CREATE DATABASE sonarqube OWNER sonar;
grant all privileges on DATABASE sonarqube to sonar;
\q
exit

###################################################################
#Increase Limits
#Paste the below values at the bottom of the file
sudo vim /etc/security/limits.conf
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096
#Exit And Save

###################################################################
#Paste the below values at the bottom of the file
sudo vim /etc/sysctl.conf
vm.max_map_count = 262144
#Exit And Save

###################################################################
#Reboot to set the new limits
sudo reboot

###################################################################
#Install Sonarqube
sudo wget wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.5.0.89998.zip
sudo apt install unzip
sudo unzip sonarqube-10.5.0.89998.zip -d /opt
sudo mv /opt/sonarqube-10.5.0.89998/ /opt/sonarqube
sudo groupadd sonar
sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
sudo chown sonar:sonar /opt/sonarqube -R

###################################################################
#Update Sonarqube properties with DB credentials
#Find and replace the below values, you might need to add the sonar.jdbc.url 
sudo vim /opt/sonarqube/conf/sonar.properties

sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
#Exit And Save

###################################################################
#Create service for Sonarqube
#Paste the below into the file

sudo vim /etc/systemd/system/sonar.service

[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
#Exit And Save

###################################################################
#Start Sonarqube and Enable service
sudo systemctl start sonar
sudo systemctl enable sonar
sudo systemctl status sonar
sudo tail -f /opt/sonarqube/logs/sonar.log

###################################################################
#Access the Sonarqube
http://<IP_ADDRESS>:9000
exit

```
## AWS CLI And GCP installation (Optional) | If needed
 - [Setup GCP CLI](https://cloud.google.com/sdk/docs/install)
 - [Setup AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
## Setup Jenkins And SonarQube 

 - [Setup Jenkins (DigitalOcean Link)](https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-ubuntu-22-04)

 - [Setup SonarQube (DigitalOcean Link)](https://www.digitalocean.com/community/tutorials/how-to-ensure-code-quality-with-sonarqube-on-ubuntu-18-04)


## Install Plugins on Jenkins

- Eclipse Temurin installerVersion
- Maven Integration
- Pipeline Maven Integration
- OWASP Dependency-Check
- SonarQube Scanner
- Build Timestamp
- Git Push
- Docker
- Docker Commons
- Docker Pipeline

## If Necessary (Optional)
- Kubernetes Client API
- Kubernetes Credentials
- Kubernetes
- Kubernetes CLI
- Google Kubernetes Engine
- Google Cloud Platform SDK :: Auth
- AWS Credentials
- Amazon Web Services SDK :: All


#
# Setup credentials on jenkins

### Generate tokens in SonarQube and GitHub, and then copy them.


![Sonarqube token copy](https://github.com/Kishanrampure/DevOps-Boardgame-Project/assets/121344253/f1eb3e05-8535-466a-bf0b-64c3a3a07fef)


##

![Github Token Copy](https://github.com/Kishanrampure/DevOps-Boardgame-Project/assets/121344253/bb465b19-cff8-4917-8b16-46c44ead354e)

###
#### Click on + Add credentials
###

![Jenkins Create Cred](https://github.com/Kishanrampure/DevOps-Boardgame-Project/assets/121344253/d1040adb-4286-4b9e-8c56-ee738dc87f84)


## 

![Jenkins cred page](https://github.com/Kishanrampure/DevOps-Boardgame-Project/assets/121344253/548d2553-285d-42a4-a8ae-e5e6e428ea4e)


## Setup Jenkins Tools

| Tool | Name | Versions ~  |
| ----------------- | | ----------------- |------------------------------------------------------------------ |
| JDK | jdk |Install from adoptium.net - 17 |
| SonarQube Scanner installations | sonar-scanner |Install automatically - 5.0.1 |
| Maven installations | M3 |Install from Apache - 3.9.6 |
| Dependency-Check installations | OWAPS |Install from github.com - 9.1.0 |

## Configure SonarQube servers in jenkins

#### Go to Manage Jenkins -> System -> SonarQube servers
##
![Sonarqube jenkins install setup](https://github.com/Kishanrampure/DevOps-Boardgame-Project/assets/121344253/eeb10e0e-062f-4f06-a8df-370d1c9b5613)

Modify build timestamp formatting
##
![Build Time Stamp](https://github.com/Kishanrampure/DevOps-Boardgame-Project/assets/121344253/a9b321d1-d76a-4109-a6e2-c273361b1a55)


#
## Repository Configure in Jenkins Pipeline
> Create a Jenkins Pipeline Job  
![Jenkins repo setup](https://github.com/Kishanrampure/DevOps-Boardgame-Project/assets/121344253/71de4f05-99dc-4834-923a-0dbc4dcb6b11)
###
- Build the Jenkins Pipeline
![Jenkins Full Pipeline](https://github.com/Kishanrampure/DevOps-Boardgame-Project/assets/121344253/a6b9912b-d151-427d-87b7-d22764025f5a)



# Deploy Application to ArgoCD (GCP || EKS) 


# Steps
- [x] Create a kubernetes cluster on GKE || EKS.
- [x] Setup Connection to created GKE || EKS cluster in with your local machine or cloud shell.
 - [Install kubectl and configure cluster access  - GCP LINK](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl#apt)
  - [Installing or updating kubectl  - AWS LINK](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html#:~:text=To%20install%20or%20update%20kubectl%20on%20Windows,Kubernetes%20version%20from%20Amazon%20S3.&text=amd64%2Fkubectl.exe-,(Optional)%20Verify%20the%20downloaded%20binary%20with%20the%20SHA%2D256,cluster's%20Kubernetes%20version%20for%20Windows.)
  
- [x] Install ArgoCD and Access.
    ```sh
    # install ArgoCD in k8s
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

    # access ArgoCD UI
    kubectl get svc -n argocd
    kubectl port-forward svc/argocd-server 8080:443 -n argocd

    # login with admin user and below token (as in documentation):
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo

    #Git Clone Repo
    git clone git clone https://github.com/Kishanrampure/DevOps-Petclinic-Project.git

    #Change Dir
    cd cd DevOps-Petclinic-Project/cicd/
    
    # Ensure the application.yaml and Deploy, service file are in the same directory
    kubectl apply -f application.yaml

    ```

- [x] Check status of the deployment.
    ```sh
    kubectl get deploy -n petclinic
    ```
- [x] Check status of service.
    ```sh
    kubectl get svc -n petclinic
    ```
- [x] Check the external IP of the service.

- [x] Verify and test the deployed application for functionality.


### Cleanup
```sh
kubectl delete -f deploy.yml
kubectl delete -f service.yml
kubectl delete -f application.yml
```

> Delete your GKE || EKS Cluster from Console.

###

## Containerising Pet Clinic app using Docker

This is a [dockerized version of the original app](https://github.com/spring-projects/spring-petclinic) published by Spring Boot community. 


## Running PetClinic app locally

Petclinic is a [Spring Boot](https://spring.io/guides/gs/spring-boot) application built using Maven. It is an application designed to show how the Spring stack can be used to build simple, but powerful database-oriented applications. The official version of PetClinic demonstrates the use of Spring Boot with Spring MVC and Spring Data JPA.

## How it works?

Spring boot works with MVC (Model-View-Controller) is a pattern in software design commonly used to implement user interfaces, data and control logic. It emphasizes a separation between business logic and its visualization. This "separation of concerns" provides a better division of labor and improved maintenance.We can work with the persistence or data access layer with [spring-data](https://spring.io/projects/spring-data) in a simple and very fast way, without the need to create so many classes manually. Spring data comes with built-in methods below or by default that allow you to save, delete, update and/or create.


## Getting Started


```
git clone https://github.com/dockersamples/spring-petclinic.git
cd spring-petclinic
./mvnw package
java -jar target/*.jar
```

You can then access petclinic here: http://localhost:8080/

<img width="625" alt="image" src="https://user-images.githubusercontent.com/313480/179161406-54a28200-d52e-411f-bfbe-463cf64b64b3.png">

The applications allows you to perform the following set of functions:

- Add Pets
- Add Owners
- Finding Owners
- Finding Veterinarians
- Exceptional handling


Or you can run it from Maven directly using the Spring Boot Maven plugin. If you do this it will pick up changes that you make in the project immediately (changes to Java source files require a compile as well - most people use an IDE for this):

```
./mvnw spring-boot:run
```

> NOTE: Windows users should set `git config core.autocrlf true` to avoid format assertions failing the build (use `--global` to set that flag globally).

> NOTE: If you prefer to use Gradle, you can build the app using `./gradlew build` and look for the jar file in `build/libs`.

## Building a Container

```
 docker build -t petclinic-app . -f Dockerfile
```

## Multi-Stage Build

```
 docker build -t petclinic-app . -f Dockerfile.multi
```

## Using Docker Compose

```
 docker-compose up -d
```



## References

- [Building PetClinic app using Dockerfile](https://docs.docker.com/language/java/build-images/)


