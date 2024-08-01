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

Grant Jenkins user and ubuntu user permission to docker deamon.

```
sudo su - 
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```

As we installed many tools, its better to restart the server

```
sudo reboot now
```

**Note: ** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as show below.

- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules as shown in the image (you can just allow 8080).

<img width="1187" alt="Screenshot 2023-02-01 at 12 42 01 PM" src="https://user-images.githubusercontent.com/43399466/215975712-2fc569cb-9d76-49b4-9345-d8b62187aa22.png">


### Login to Jenkins using the below URL:

`http://<ec2-instance-public-ip-address>:8080`  [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

Note: If you are not interested in allowing `All Traffic` to your EC2 instance
   -  1. Delete the inbound traffic rule for your instance
   -  2. Edit the inbound traffic rule to only allow custom TCP port `8080`

After you login to Jenkins, 
      - Run the command to copy the Jenkins Admin Password - `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
      - Enter the Administrator password
      
<img width="1291" alt="Screenshot 2023-02-01 at 10 56 25 AM" src="https://user-images.githubusercontent.com/43399466/215959008-3ebca431-1f14-4d81-9f12-6bb232bfbee3.png">

Click on Install suggested plugins

<img width="1291" alt="Screenshot 2023-02-01 at 10 58 40 AM" src="https://user-images.githubusercontent.com/43399466/215959294-047eadef-7e64-4795-bd3b-b1efb0375988.png">

Wait for the Jenkins to Install suggested plugins

<img width="1291" alt="Screenshot 2023-02-01 at 10 59 31 AM" src="https://user-images.githubusercontent.com/43399466/215959398-344b5721-28ec-47a5-8908-b698e435608d.png">

Create First Admin User or Skip the step [If you want to use this Jenkins instance for future use-cases as well, better to create admin user]

<img width="990" alt="Screenshot 2023-02-01 at 11 02 09 AM" src="https://user-images.githubusercontent.com/43399466/215959757-403246c8-e739-4103-9265-6bdab418013e.png">

Jenkins Installation is Successful. You can now starting using the Jenkins 

<img width="990" alt="Screenshot 2023-02-01 at 11 14 13 AM" src="https://user-images.githubusercontent.com/43399466/215961440-3f13f82b-61a2-4117-88bc-0da265a67fa7.png">

Install the Required plugins in Jenkins

   - Log in to Jenkins.
   - Go to Manage Jenkins > Manage Plugins.
   - In the Available tab, search for `Docker Pipeline`,`CloudBees Docker Build and Publish plugin`,`Docker plugin`,`docker-build-step
     `,`SonarQube Scanner`.
   - Select the plugins and click the Install button.
   - Restart Jenkins after the plugin is installed. `http://<ec2-instance-public-ip-address>:8080/restart`
   
<img width="1392" alt="Screenshot 2023-02-01 at 12 17 02 PM" src="https://user-images.githubusercontent.com/43399466/215973898-7c366525-15db-4876-bd71-49522ecb267d.png">

Wait for the Jenkins to be restarted.

### Install SonarQube and Trivy:

   - Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.
        
       sonarqube
       ```
       docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
       ```

       To access: 
        
        publicIP:9000 (by default username & password is admin)
        
        To install Trivy:
     
        ```
        sudo apt-get install wget apt-transport-https gnupg lsb-release
        wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
        echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
        sudo apt-get update
        sudo apt-get install trivy        
        ```

         to scan image using trivy
     
        ```
        trivy image <imageid>
        ```
        
        
### Integrate SonarQube and Configure:

   - Integrate SonarQube with your CI/CD pipeline.
   - Configure SonarQube to analyze code for quality and security issues.
