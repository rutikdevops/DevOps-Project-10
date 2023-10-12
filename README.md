# Deploying a Pet Clinic Java Based Application.
# DevOps-Project-10

# Project Blog link :-

# Project Overview :-
# Project Steps :-
Step 1 — Create an Ubuntu T2 Large Instance
Step 2 — Install Jenkins, Docker and Trivy. Create a Sonarqube Container using Docker.
Step 3 — Install Plugins like JDK, Sonarqube Scanner, Maven, OWASP Dependency Check,
Step 4 — Create a Pipeline Project in Jenkins using Declarative Pipeline
Step 5 — Install OWASP Dependency Check Plugins
Step 6 — Docker Image Build and Push
Step 7 — Deploy image using Docker
Step 8 — Access the Real World Application
Step 9 — Terminate the AWS EC2 Instance


Now, lets get started and dig deeper into each of these steps :-

Step 1 — Launch an AWS T2 Large Instance. Use the image as Ubuntu. You can create a new key pair or use an existing one. Enable HTTP and HTTPS settings in the Security Group.

Step 2 — Install Jenkins, Docker and Trivy



2A — To Install Jenkins
Connect to your console, and enter these commands to Install Jenkins


```bash
vi jenkins.sh
```

```bash
#!/bin/bash
sudo apt update -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
                  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
                  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
                              /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```


```bash
sudo chmod 777 jenkins.sh
./jenkins.sh
```
- Once Jenkins is installed, you will need to go to your AWS EC2 Security Group and open Inbound Port 8080, since Jenkins works on Port 8080.
- Now, grab your Public IP Address
```bash
<EC2 Public IP Address:8080>
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

2B — Install Docker
```bash
vi docker.sh
```

```bash
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock
docker ps
```

```bash
chmod 700 docker.sh
./docker.sh
systemctl start docker
systemctl enable docker
```
- After the docker installation, we create a sonarqube container (Remember added 9000 port in the security group)

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

2C — Install Trivy
```bash
vi trivy.sh
```

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```
- Next, we will login to Jenkins and start to configure our Pipeline in Jenkins



Step 3 — Install Plugins like JDK, Sonarqube Scanner, Maven, OWASP Dependency Check,
3A — Install Plugin
Goto Manage Jenkins →Plugins → Available Plugins →
Install below plugins
1 → Eclipse Temurin Installer (Install without restart)
2 → SonarQube Scanner (Install without restart)
3B — Configure Java and Maven in Global Tool Configuration
Goto Manage Jenkins → Tools → Install JDK and Maven3 → Click on Apply and Save
3C — Create a Job
Label it as Real-World CI-CD, click on Pipeline and Ok.


Enter this in Pipeline Script
```bash
pipeline {
    agent any 
      
    tools{
        jdk 'jdk11'
        maven 'maven3'
    }
    
    stages{
        stage('clean workspace'){
             steps{
                 cleanWs()
             }
         }
        stage("Git Checkout"){
            steps{
                git 'https://github.com/Aj7Ay/amazon-eks-jenkins-terraform-aj7.git'
            }
        }
        
        stage("Maven Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
    }
}
```



Step 4 — Configure Sonar Server in Manage Jenkins
Grab the Public IP Address of your EC2 Instance, Sonarqube works on Port 9000 , sp <Public IP>:9000. Goto your Sonarqube Server. Click on Administration → Security → Users → Click on Tokens and Update Token → Give it a name → and click on Generate Token

<img width="482" alt="image" src="https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/c3bae880-2c47-46f6-98dc-6877b9e5a233">

Click on Update Token
<img width="482" alt="image" src="https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/3dff25d3-fc8f-4b57-9ab1-d93cbae30e40">

Copy this Token ,
Goto Dashboard → Manage Jenkins → Credentials → Add Secret Text. It should look like this






















