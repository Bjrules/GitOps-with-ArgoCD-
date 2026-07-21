# GitOps-with-ArgoCD-
Implementing GitOps Practices using ArgoCD 
It is Important to know the difference between Traditional CICD and GitOps and the Advantages of using the both of them.

 In this project, I successfully set up a complete CI (Continous Integration) pipeline using various DevOps tools on an AWS EC2 and EKS. I started by configuring essential services like Jenkins, Docker, SonarQube and Nexus to ensure seamless code integration, quality checks and artifact management.I made sure that Jenkins Continous Integration Pipeline is done on this branch "CI-main" of this repo https://github.com/Bjrules/GitOps-with-ArgoCD-.git and it had to update the "CD-GitOps" branch of this very Repository with the latest Docker build version with tagging it correspondingly. Then I setup ArgoCD on EKS cluster, for deploying applications on Kubernetes with it's pull-base technology(from the GitOps branch) for automated reconciliation of both Desired-state and Actual-state(EKS-Cluster). Thus, a seamless automation for deployment using Git as the "single point of truth" and that was how I built a robust and scalable environment for continuous delivery. 
> NB: GitOps with ArgoCD kind of automation wrks for both Infrastructure and Application.

#### Note: Jenkins now works on jdk21 but since I am deploying a java SpringBoot application that works on jdk17, I had to install the both of them, but if I do ` sudo update-alternatives --config java`

