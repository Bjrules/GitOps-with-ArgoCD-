# GitOps-with-ArgoCD-
Implementing GitOps Practices using ArgoCD 
It is Important to know the difference between Traditional CICD and GitOps and the Advantages of using the both of them.

 In this project, I successfully set up a complete CI (Continous Integration) pipeline using various DevOps tools on an AWS EC2 and EKS. I started by configuring essential services like Jenkins, Docker, SonarQube and Nexus to ensure seamless code integration, quality checks and artifact management.I made sure that Jenkins Continous Integration Pipeline is done on this branch "CI-main" of this repo https://github.com/Bjrules/GitOps-with-ArgoCD-.git and it had to update the "CD-GitOps" branch of this very Repository with the latest Docker build version with tagging it correspondingly. Then I setup ArgoCD on EKS cluster, for deploying applications on Kubernetes with it's pull-base technology(from the GitOps branch) for automated reconciliation of both Desired-state and Actual-state(EKS-Cluster). Thus, a seamless automation for deployment using Git as the "single point of truth" and that was how I built a robust and scalable environment for continuous delivery. 
> NB: GitOps with ArgoCD kind of automation wrrks for both Infrastructure and Application Deployments.

 Note: Jenkins now works on jdk21 but since I am deploying a java SpringBoot application that works on jdk17, I had to install the both of them, but if I do ` sudo update-alternatives --config java` and select jdk17, jenkins Server then crashes almost irrevokably. so the option is to set jdk 17 `JAVA_HOME` and `PATH` in the `environment` section of the pipeline see screenshot below.
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_170958.png)


## Project Steps.
Set Up the Following EC2 of t2.medium each
1. Infra Server (for EKS Setup)
2. Jenkins Server (For CI pipeline)
3. SonarQube Server (for SAST)
4. Nexus (for Artifact Storage and versioning)

### Install and Configure JENKINS

```
sudo apt update
sudo apt install fontconfig openjdk-21-jre-headless -y
java -version   # confirm it shows 21

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y

sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins

```


**DOCKER INSTALLATION**
```

# Remove any conflicting old packages first
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

sudo systemctl enable docker
sudo systemctl start docker
sudo docker run hello-world
```



Install Docker on Jenkins, SonarQube and Nexus Servers as We will be building and pushing docker images from jenkins Server While SonarQube and Nexus will be run as Docker container from Docker images on their respective machines. 

**ON JENKINS SERVER**
- After installation of Docker
- `sudo usermod -aG docker $USER`
- `sudo usermod -aG jenkins` So thats jenkins as a user service account can run Docker
- `newgrp docker`

**ON SONARQUBE SERVER**
- After installation of Docker
- `sudo usermod -aG docker $USER`
- `newgrp docker`
- `docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community`
- The head up to browser `http://<Public-IP-Address>/9000` to configure

**ON NEXUS SERVER**
- After installation of Docker
- `sudo usermod -aG docker $USER`
- `newgrp docker`
- `docker run -d --name nexus -p 8081:8081 sonatype/nexus3`
- The head up to browser `http://<Public-IP-Address>/8081` to configure




> Install Jenkins plugins
![alt text](IMG-SCREENSHOTS/Screenshot_20260719_131004.png)
---
![alt text](IMG-SCREENSHOTS/Screenshot_20260719_135531.png)
Configure Credentials for Docker login, Github Login and Sonarqube server integration in Jenkins
![alt text](IMG-SCREENSHOTS/Screenshot_20260720_173207.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260719_142129.png)
Configure Sonarqube Server URL with the security token in Jenkin
![alt text](IMG-SCREENSHOTS/Screenshot_20260719_142345.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260719_142604.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_013401.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_013416.png)
 Also go to Manage jenkins>Tools to configure `sonar-scanner` for jenkins 
 ![alt text](IMG-SCREENSHOTS/Screenshot_20260720_171601.png)

> SonarQube server showing the Analysis of the Bankapp
![alt text](IMG-SCREENSHOTS/Screenshot_20260719_155343.png)
---

### Setup Nexus Artifact Server

![alt text](IMG-SCREENSHOTS/Screenshot_20260720_210359.png)
Config File Management for Nexus Configurations  so as to modify the credentials of the `maven-releases` and `maven-snapshots` with the username and passwords accordingly
![alt text](IMG-SCREENSHOTS/Screenshot_20260719_163016.png)
Also The pom.xml file in the `CI-main` branch of this repo need to be updated with the url of the Nexus server while pointing to maven-releases `http://54.234.21.34:8080/repository/maven-releases` and `http://54.234.21.34:8080/repository/maven-snapshot`  