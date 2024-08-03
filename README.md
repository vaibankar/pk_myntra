# NodeJS web application deploy on Kind


This is a simple NodeJS web application that can be built using npm. NodeJS dependencies are handled using the package.json at the root directory of the repository.

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
   - Prometheus
   - Grafana

### Configuring Jenkins server

Pre-Requisites:
         
   -  Java-17
   -  Jenkins

Install Java

```shell
sudo apt update -y
sudo apt install openjdk-17-jre-headless -y
java -version
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

Install NodeJS

```shell
curl -sL https://deb.nodesource.com/setup_16.x | sudo bash -
sudo apt install nodejs -y
```


**Note: ** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as show below.

- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups -> Edit inbound rules
- add 8080 Port for jenkins 



### Login to Jenkins using the below URL:

`http://<ec2-instance-public-ip-address>:8080`  [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

   - Edit the inbound traffic rule to only allow custom TCP port `8080`

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
   - In the Available tab, search for
        - NodeJs
        - SonarQube Scanner
        - CloudBees Docker Build and Publish plugin, Docker Pipeline, Docker plugin, docker-build-step
   - Select the plugins and click the Install button.
   - Restart Jenkins after the plugin is installed. `http://<ec2-instance-public-ip-address>:8080/restart`
   
<img width="1392" alt="Screenshot 2023-02-01 at 12 17 02 PM" src="https://user-images.githubusercontent.com/43399466/215973898-7c366525-15db-4876-bd71-49522ecb267d.png">

Wait for the Jenkins to be restarted.


## Install Docker:


- Set up Docker on the EC2 instance:

```
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl status docker
```


Grant Jenkins user and ubuntu user permission to docker deamon.

```
sudo su - 
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```

Once you are done with the above steps, it is better to restart Jenkins.

```
http://<ec2-instance-public-ip>:8080/restart
```


### Install SonarQube:

   - Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.
        
       sonarqube
       ```
       docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
       ```

       To access: 
        
        publicIP:9000 (by default username & password is admin)
        
        
        
        
### Integrate SonarQube and Configure:

   - Integrate SonarQube with your CI/CD pipeline.
   - Configure SonarQube to analyze code for quality and security issues.


**Note: ** By default, SonarQube will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 9000 in the inbound traffic rules as show below.

- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups -> Edit inbound rules
- add 9000 Port for SonarQube 



### Login to SonarQube using the below URL:

`http://<ec2-instance-public-ip-address>:9000`  [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

   
   - Edit the inbound traffic rule to only allow custom TCP port `9000`

Hurray !! Now you can access the `SonarQube Server` on `http://<ec2-instance-public-ip-address>:9000` 
   
![Screenshot (199)](https://user-images.githubusercontent.com/129657174/230658262-0a0c8d3a-312d-4423-84e5-a7a1efa9fc68.png)

Login using username: admin, Passsword: admin and Change the password
   
![Screenshot (200)](https://user-images.githubusercontent.com/129657174/230658269-22ebd5d8-3f97-4a47-8155-57236a95b055.png)

Now at the right top corner click profile icon ->  My Account -> Security, Under Generate Token give a name and click Generate and copy the Token.
   
![Screenshot (201)](https://user-images.githubusercontent.com/129657174/230658498-75e5a06e-512e-4665-bd81-68d7a87d3469.png)
![Screenshot (202)](https://user-images.githubusercontent.com/129657174/230658495-a4ee14e9-df19-4bfa-8cec-0b9ccc3abb76.png)


### Configuring Credentials on Jenkins

For SonarQube

   -  Go to Manage Jenkins > Manage Credentials > System > global > Add Credentials
   -  Select Kind as Secret text
   -  Copy the Sonarqube Token in Secret box and give name as sonarqube in ID
   -  Click Save


For DockerHub

   -  Now go to [DockerHub](https://hub.docker.com/) create a user and create a new repository with name 'Myntra'
   -  Go to Manage Jenkins > Manage Credentials > System > global > Add Credentials
   -  Select Kind as Username and Password
   -  Give the username and password and give your `Dockerhub username` and `Dockerhub Password` id is `docker`.
   -  Click Save

### Install Trivy:

To install Trivy:
     
        ```
        sudo apt-get install wget apt-transport-https gnupg lsb-release
        wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
        echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
        sudo apt-get update
        sudo apt-get install trivy        
        ```

        

To scan image using trivy:
        
        ```
        trivy image <imageid>
        ```


### Install Kind:

- Set up Kind on the EC2 instance:

     ```

     ```

<img width="400" hight="400" alt="demo" src="https://i.imgur.com/kx7YEYf.jpeg">



