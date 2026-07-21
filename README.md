# GitOps-with-ArgoCD-
Implementing GitOps Practices using ArgoCD 


 In this project, I successfully set up a complete CI (Continous Integration) pipeline using various DevOps tools on an AWS EC2 and EKS. I started by configuring essential services like Jenkins, Docker, SonarQube and Nexus to ensure seamless code integration, quality checks and artifact management.I made sure that Jenkins Continous Integration Pipeline is done on this branch "CI-main" of this repo https://github.com/Bjrules/GitOps-with-ArgoCD-.git and it had to update the "CD-GitOps" branch of this very Repository with the latest Docker build version with tagging it correspondingly. Then I setup ArgoCD on EKS cluster, for deploying applications on Kubernetes with it's pull-base technology(from the GitOps branch) for automated reconciliation of both Desired-state and Actual-state(EKS-Cluster) thus, a seamless automation for deployment using Git as the "single point of truth" and that was how I built a robust and scalable environment for continuous delivery. 
> NB: GitOps with ArgoCD kind of automation works for both Infrastructure and Application Deployments.
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_224543.png)

 Note: Current release of Jenkins now works on jdk21 but since I am deploying a java SpringBoot application that works on jdk17, I had to install the both of them, but if I do ` sudo update-alternatives --config java` and select jdk17, jenkins Server then crashes almost irrevokably. so the option is to set jdk 17 `JAVA_HOME` and `PATH` in the `environment` section of the pipeline see screenshot below. 
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_170958.png)


## Project Steps.
Set Up the Following EC2 of t2.medium each
1. Infra Server (for EKS Setup)
2. Jenkins Server (For CI pipeline)
3. SonarQube Server (for SAST)
4. Nexus (for Artifact Storage and versioning)

### JENKINS INSTALLATION
`sudo apt update`
Install jdk17 -> `sudo apt install openjdk-17-jre-headless`
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
![alt text](IMG-SCREENSHOTS/Screenshot_20260720_210359.png)



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

Screenshot showing Deployed after jenkins stage(Buuld and Publish Artifact to Nexus) is succesful`sh 'mvn deploy -DskipTests=true'`
 ![alt text](IMG-SCREENSHOTS/Screenshot_20260720_225820.png)
 ![alt text](IMG-SCREENSHOTS/Screenshot_20260721_024803.png)

**ON INFRA SERVER**
install terraform 
- `sudo snap install terraform --classic`
see EKS-Terraform-main Directory in this Repo (Clone it, cd into it, Configure the variables and `terraform init` `terraform plan` and `terraform apply --auto-approve` to bootstrap Kubernetes )

Install AWS CLI & CONFIGURE (Get the Access and Secret key from AWS)
```
IMG-SCREENSHOTS/Screenshot_20260721_040525.png
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws configure

```
Install KUBE CTL
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client

```
Install EKS CTL
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

```
connect to the cluster
`awk eks --region us-east-1 update-config --name bj-cluster`
confirm `kubectl get nodes`
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_041613.png)

**SET UP ARGOCD**
create a namespace secifically for ArgoCD
`kubectl create namespace argocd`
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_041927.png)

Install ArgoCD and all of it's other services into the namespace created above in kubernetes cluster using the command below
`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_042053.png)
Verify the ArgoCD Installation:
`kubectl get pods -n argocd`
`kubectl get all -n argocd`
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_042145.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_042535.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_042557.png)

Since ArgoCD's server is ClusterIP by default, I need to change the  service: type: LoadBalancer thus:

```
kubectl edit svc argocd-server -n argocd

```
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_042739.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_042814.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_042858.png)

Then

hit the browser with the public DNS IP or ADDRESS
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_043149.png)

while the username is `admin` Get the Password using:

`kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d`

![alt text](IMG-SCREENSHOTS/Screenshot_20260721_043459.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_043439.png)

Configure ArgoD to connect to my Connect to CD-GitOps branch of my Repo (Single point of truth)

![alt text](IMG-SCREENSHOTS/Screenshot_20260721_044942.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_045913.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_050157.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_050625.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_050730.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_050823.png)

![alt text](IMG-SCREENSHOTS/Screenshot_20260721_051052.png)

Yipee.... It works !!!!! Fully Automated.............
> The interface on my Java SpringBoot Bank Application

![alt text](IMG-SCREENSHOTS/Screenshot_20260721_051434.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_051450.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_051540.png)

Lets Modify to see if it will work.

![alt text](IMG-SCREENSHOTS/Screenshot_20260721_051949.png)

It Works! synced having configured auto-sync....

![alt text](IMG-SCREENSHOTS/Screenshot_20260721_052016.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_052630.png)

Proctove Measure incase to aid AutoSync incase of server freeze, i ha dto configure webhook in Github as well.

![alt text](IMG-SCREENSHOTS/Screenshot_20260721_052746.png)
![alt text](IMG-SCREENSHOTS/Screenshot_20260721_053156.png)

A Screenshot of my Workspace in Jenkins.

# THANK YOU.


- To clone the CI Repo `git clone -b CI-main https://github.com/Bjrules/GitOps-with-ArgoCD-.git`
- To clone the CD Repo `git clone -b CD-GitOps https://github.com/Bjrules/GitOps-with-ArgoCD-.git`