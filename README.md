# Myntra Clone NodeJS web application deploy on Kind
<img width="960" alt="Home" src="https://user-images.githubusercontent.com/96285027/235316694-52148226-09f7-494f-9012-6dec10d2980c.png">

This is a simple Myntra NodeJS web application that can be built using npm. Myntra-NodeJS dependencies are handled using the package.json at the root directory of the repository.

Here are the step-by-step details to set up an end-to-end Jenkins pipeline for a NodeJS application using SonarQube, Argo CD and Kind:

Prerequisites:

   -  NodeJS application code hosted on a Git repository
   -  Jenkins server
   -  Kind cluster
   -  Argo CD
      
Tools Required:
    
   - Java
   - Git
   - npm
   - Jenkins
   - SonarQube
   - Docker
   - Kind
   - ArgoCD

### Configuring Jenkins server

Pre-Requisites:
         
   -  Java
   -  Git
   -  npm
   -  Docker
   -  Jenkins

Install Java

```shell
sudo apt update -y
sudo apt install openjdk-17-jre-headless -y
java -version
```

Install Git

```shell
sudo apt install git -y
git --version
```

Install NodeJS

```shell
curl -sL https://deb.nodesource.com/setup_16.x | sudo bash -
sudo apt install nodejs -y
```

Install Docker

```
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl status docker
```

Install Jenkins

```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
```
