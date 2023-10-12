# Deploying a Pet Clinic Java Based Application.
# DevOps-Project-10

# Project Blog link :-

# Project Overview :-
# Project Steps :-
- Step 1 — Create an Ubuntu T2 Large Instance
- Step 2 — Install Jenkins, Docker and Trivy. Create a Sonarqube Container using Docker.
- Step 3 — Install Plugins like JDK, Sonarqube Scanner, Maven, OWASP Dependency Check,
- Step 4 — Create a Pipeline Project in Jenkins using Declarative Pipeline
- Step 5 — Install OWASP Dependency Check Plugins
- Step 6 — Docker Image Build and Push
- Step 7 — Deploy image using Docker
- Step 8 — Access the Real World Application
- Step 9 — Terminate the AWS EC2 Instance


# Step 1 :-
-  Launch an AWS T2 Large Instance. Use the image as Ubuntu. You can create a new key pair or use an existing one. Enable HTTP and HTTPS settings in the Security Group.
<img width="948" alt="image" src="https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/8f7aa15f-43e5-4247-b0c9-bae2e9b89efb">

# Step 2 :-
- Install Jenkins, Docker and Trivy

- 2A — To Install Jenkins
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

- 2B — Install Docker
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

- 2C — Install Trivy
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
```bash
chmod 700 trivy.sh
./trivy.sh
```
- Next, we will login to Jenkins and start to configure our Pipeline in Jenkins



# Step 3 :-
- Install Plugins like JDK, Sonarqube Scanner, Maven, OWASP Dependency Check,
- 3A — Install Plugin
- Goto Manage Jenkins →Plugins → Available Plugins →
- Install below plugins
1 → Eclipse Temurin Installer (Install without restart)
2 → SonarQube Scanner (Install without restart)
- 3B — Configure Java and Maven in Global Tool Configuration
- Goto Manage Jenkins → Tools → Install JDK and Maven3 → Click on Apply and Save
![image](https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/af0f9860-1009-4ef1-9416-8d36fdff09c4)
![image](https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/9d25fa1b-4559-4ed8-9f39-b6360d8d628c)

- 3C — Create a Job --> New Item --> Label it as Real-World CI-CD --> Pipeline --> OK
![image](https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/6bfa5926-bbc8-4c7f-8bba-0df71e70a90a)

- Enter this in Pipeline Script
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



# Step 4 :-
- Configure Sonar Server in Manage Jenkins
- Grab the Public IP Address of your EC2 Instance, Sonarqube works on Port 9000 , sp <Public IP>:9000. Goto your Sonarqube Server. Click on Administration → Security → Users → Click on Tokens and Update Token → Give it a name → and click on Generate Token --> lick on Generate Token --> Copy this Token
- ![image](https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/3173b9cb-ad23-47ac-ac6a-79f4e0889c87)
- ![image](https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/b8d96e51-3130-4bba-afab-e26ec390a4f4)

- Goto Dashboard → Manage Jenkins → Credentials → Global Credentials --> Add Credential --> Add Secret Text
- ![image](https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/80a76ad2-ebc6-468d-9ac5-d1f5ba42c2fc)

- Now, goto Dashboard → Manage Jenkins → Configure System --> SonarQube installations
<img width="946" alt="image" src="https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/3c11ed4e-ebc7-4e3b-a30b-bbad6ec35732">

- SonarQube --> Administration --> Configurations --> Webhooks --> Create
<img width="959" alt="image" src="https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/fd64c1a6-c0ac-42c9-9ed9-48451c94f227">

- Add this script in pipeline :-
```bash
        stage("Sonarqube Analysis "){
            steps{
                script{
                withSonarQubeEnv(credentialsId: 'Sonar-token') {
                sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        stage('Quality Gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
```

- To see the report, you can goto Sonarqube Server and goto Projects.
<img width="960" alt="image" src="https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/f012fd67-f297-41cf-849d-7a2d6cb41140">



# Step 5 :-
- Install OWASP Dependency Check Plugins
- GotoDashboard → Manage Jenkins → Plugins → OWASP Dependency-Check. Click on it and install without restart.
- First, we configured Plugin and next we have to configure Tool
- Goto Dashboard → Manage Jenkins → Tools → Dependency-Check installations
![image](https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/cfa564f3-8e3f-45b4-9a18-18b1f99028c3)


- Now goto configure → Pipeline and add this stage to your pipeline
```bash
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Build war file"){
            steps{
                sh " mvn clean install"
            }
        }
```


# Step 6 :-
- Docker Image Build and Push
- We need to install Docker tool in our system, Goto Dashboard → Manage Plugins → Available plugins → Search for Docker and install these plugins
- Docker, Docker Commons, Docker Pipeline, Docker API, docker-build-step

- Now, goto Dashboard → Manage Jenkins → Tools → Docker installations
![image](https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/ae0aede3-cc05-4395-8fc9-4d25d5df8899)

- Add DockerHub Username and Password under Global Credentials --> Dashboard -->Manage Jenkins -->Credentials -->System -->Global credentials (unrestricted)
<img width="526" alt="image" src="https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/c14e6297-2fab-4ed5-a35e-2f832f7a8489">

- Add this stage in Pipeline Script
```bash
stage("Docker Build & Push"){
    steps{
        script{
        withDockerRegistry(credentialsId: 'docker' , toolName: 'docker') {
                sh "docker build -t petclinic1 ."
                sh "docker tag petclinic1 rutikdevops/pet-clinic123:latest "
                sh "docker push rutikdevops/pet-clinic123:latest "
            }
        }
    }
}
```

- Now, when you do
```bash
docker images
```

- Docker Containers
```bash
docker ps
```

- When you log in to Dockerhub, you will see a new image is created
<img width="944" alt="image" src="https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/f2d0aa93-2247-4c51-b8b0-ee25ca707c50">


# Step 7 :-
- Deploy image using Docker
- Add this stage to your pipeline syntax
```bash
stage("Deploy Using Docker"){
    steps{
        sh " docker run -d --name pet1 -p 8082:8080 rutikdevops/pet-clinic123:latest "
    }
}
```
![image](https://github.com/rutikdevops/DevOps-Project-10/assets/109506158/9d3e4049-db1a-4b9c-a24d-9cd741cd1ea7)



# Step 8 :-
- Terminate the AWS EC2 Instance


# Project Reference :-
- https://youtu.be/Rj9oQHC12c4?feature=shared






