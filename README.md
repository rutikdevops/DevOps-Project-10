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










