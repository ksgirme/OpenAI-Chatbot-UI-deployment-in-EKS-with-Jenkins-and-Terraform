# 🤖 OpenAI Chatbot UI Deployment in EKS using Jenkins and Terraform

![Project Flow](.public/devops.jpg)

This project demonstrates a complete **DevSecOps pipeline** to deploy a **ChatGPT-powered Chatbot UI** on **Amazon EKS** using **Jenkins** for CI/CD and **Terraform** for infrastructure provisioning. The pipeline includes code quality checks, vulnerability scanning, Docker image handling, and secure container deployment.

---

## 🔧 Tech Stack

- **GitHub** – Source code management  
- **Ubuntu EC2 (t2.large / 30GB)** – Jenkins host  
- **Jenkins** – CI/CD Orchestration  
- **Node.js & npm** – Application build  
- **SonarQube** – Code quality analysis  
- **OWASP Dependency Check** – Vulnerability scanning  
- **Trivy** – File and Docker image scanning  
- **Docker** – Containerization  
- **Kubernetes (EKS)** – Deployment  
- **Terraform** – Infrastructure provisioning  

---

## 📊 Pipeline Stages

| Stage     | Description                      | Tool Used                 |
|-----------|----------------------------------|---------------------------|
| Stage 1   | Clone from GitHub                | GitHub                    |
| Stage 2   | Install Dependencies             | npm                       |
| Stage 3   | Code Quality Analysis            | SonarQube                 |
| Stage 4   | Dependency Vulnerability Scan    | OWASP Dependency Check    |
| Stage 5   | Trivy File Scan                  | Trivy                     |
| Stage 6   | Docker Build and Push            | Docker                    |
| Stage 7   | Trivy Image Scan                 | Trivy                     |
| Stage 8   | Deploy to Kubernetes             | kubectl                   |
| Stage 9   | Provision Infra (EKS)            | Terraform                 |

---

## 📁 Project Structure
.
├── Jenkinsfile
├── Dockerfile
├── sonar-project.properties
├── dependency-check.sh
├── trivy-scan.sh
├── terraform/
│ ├── main.tf
│ ├── variables.tf
│ └── outputs.tf
└── deployment/
├── deployment.yaml
├── service.yaml
└── namespace.yaml

python
Copy
Edit

---

## ⚙️ Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "your-dockerhub-username/chatbot-ui"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/your-repo/chatbot-ui.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Code Analysis - SonarQube') {
            steps {
                sh 'sonar-scanner'
            }
        }

        stage('Vulnerability Check - OWASP') {
            steps {
                sh './dependency-check.sh'
            }
        }

        stage('File Scan - Trivy') {
            steps {
                sh './trivy-scan.sh .'
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME .
                    docker push $IMAGE_NAME
                '''
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image $IMAGE_NAME"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f deployment/namespace.yaml
                    kubectl apply -f deployment/deployment.yaml
                    kubectl apply -f deployment/service.yaml
                '''
            }
        }

        stage('Provision Infrastructure - Terraform') {
            steps {
                dir('terraform') {
                    sh '''
                        terraform init
                        terraform apply -auto-approve
                    '''
                }
            }
        }
    }
}

````
## 🐳 Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY . .

RUN npm install

EXPOSE 3000

CMD ["npm", "start"]

## ⚙️ SonarQube Configuration
sonar.projectKey=ChatbotUI
sonar.projectName=Chatbot UI
sonar.projectVersion=1.0
sonar.sources=.
sonar.sourceEncoding=UTF-8

## 🔐 OWASP Dependency Check
#!/bin/bash
mkdir -p owasp
dependency-check.sh --project "Chatbot UI" --scan . --out owasp/

## 🔍 Trivy File Scan
trivy-scan.sh
#!/bin/bash
trivy fs --exit-code 0 --severity HIGH,CRITICAL $1

## ☁️ Terraform Infrastructure Code
terraform/main.tf
provider "aws" {
  region = "us-west-2"
}

resource "aws_eks_cluster" "chatbot_eks" {
  name     = "chatbot-cluster"
  role_arn = aws_iam_role.eks_cluster.arn
  # ... additional configs
}

## 📦 Kubernetes Deployment Files
deployment/namespace.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: chatbot-ui
  namespace: chatbot
spec:
  replicas: 2
  selector:
    matchLabels:
      app: chatbot-ui
  template:
    metadata:
      labels:
        app: chatbot-ui
    spec:
      containers:
      - name: chatbot-ui
        image: your-dockerhub-username/chatbot-ui
        ports:
        - containerPort: 3000

## ✅ Prerequisites
AWS CLI configured (aws configure)

Docker installed and DockerHub access

Jenkins with required plugins (Git, Pipeline, Docker, SonarQube Scanner)

Trivy and OWASP CLI tools installed

kubectl configured for EKS

Terraform >= 1.3

