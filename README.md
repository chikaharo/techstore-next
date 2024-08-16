# CICD Implementation Fullstack (Next.js + Node.js) Techstore Ecommerce App to AWS EKS
## Overview 
This document outlines the structure of a project that includes a full-stack application with distinct frontend and backend components, utilizes MongoDB as its database, and implements container orchestration with Kubernetes. Infrastructure as Code (IaC) is managed using Terraform, and a continuous integration/continuous deployment (CI/CD) process is established through Jenkins.

## Project Stucture 
- Frontend: Write by Next.js
- Backend: Write by Node.js
- Mongodb: database for this project
- Kubernetes: include frontend deploymnent, backend deployment, mongodb deployment with persistent volume and these services
- Terraform: used for provisioning cloud infrastructure
- Jenkinsfile: automate CI/CD pipeline

![Kubernetes Structure](https://github.com/user-attachments/assets/4273b037-6431-4a98-a595-747ac186e8bb)

## Demo 
<img width="1440" alt="image" src="https://github.com/user-attachments/assets/4279e116-cba6-420b-836f-9f7a6b9d928e">
<img width="1440" alt="image" src="https://github.com/user-attachments/assets/be7a8a85-77fc-4615-b11d-9016ebf6b6fa">
<img width="1431" alt="image" src="https://github.com/user-attachments/assets/2016c26f-66f3-4c82-9a7d-9b32528128c4">

## Deploy to EKS

### Create EKS cluster
Create new cluster with estcl
```
eksctl create cluster --name=EKS-1 \
                      --region=ap-south-1 \
                      --zones=ap-south-1a,ap-south-1b \
                      --without-nodegroup

eksctl utils associate-iam-oidc-provider \
    --region ap-south-1 \
    --cluster EKS-1 \
    --approve

eksctl create nodegroup --cluster=EKS-1 \
                       --region=ap-south-1 \
                       --name=node2 \
                       --node-type=t3.medium \
                       --nodes=2 \
                       --nodes-min=1 \
                       --nodes-max=3 \
                       --node-volume-size=20 \
                       --ssh-access \
                       --ssh-public-key=DevOps \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access
```

### Integration with Jenkins
- Run Jenkins on your machine or virtual machine with docker: `docker run -d -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts`
- Install plugins: Docker, Amazon Web Services, Kubernetes, Kubernetes CLI, SSH Agent
- Setup Credentials in Jenkins to integrate with Git, AWS, Kubernetes
- Create new pipeline <img width="1181" alt="image" src="https://github.com/user-attachments/assets/8154e10b-15aa-4b54-a42f-adacd36564fa">
- Enjoy automate CI/CD pipeline <img width="1390" alt="image" src="https://github.com/user-attachments/assets/5aabd64b-119c-4ed4-9f89-a5659376a34a">
