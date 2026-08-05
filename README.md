# DevOps Project Report: Automated CI/CD Pipeline for a 2-Tier Flask Application on AWS

---

# Table of Contents

1. [Project Overview](#1-project-overview)
2. [Architecture Diagram](#2-architecture-diagram)
3. [Step 1: AWS EC2 Instance Preparation](#3-step-1-aws-ec2-instance-preparation)
4. [Step 2: Install Dependencies on EC2](#4-step-2-install-dependencies-on-ec2)
5. [Step 3: Jenkins Installation and Setup](#5-step-3-jenkins-installation-and-setup)
6. [Step 4: GitHub Repository Configuration](#6-step-4-github-repository-configuration)
  - [Dockerfile](#dockerfile)
  - [docker-compose.yml](#docker-composeyml)
  - [Jenkinsfile](#jenkinsfile)
7. [Step 5: Jenkins Pipeline Creation and Execution](#7-step-5-jenkins-pipeline-creation-and-execution)
8. [Conclusion](#8-conclusion)
9. [Infrastructure Diagram](#9-infrastructure-diagram)
10. [Workflow Diagram](#10-workflow-diagram)

---

# 1. Project Overview

This project demonstrates how to implement a complete Continuous Integration and Continuous Deployment (CI/CD) pipeline for a Flask web application.

The pipeline automatically:

- Clones the source code from GitHub
- Performs static code analysis using SonarQube
- Builds a Docker image
- Pushes the Docker image to Docker Hub
- Deploys the application using Docker Compose
- Connects the application with a MySQL database

---

# 2. Architecture Diagram

```text
                    +------------------+
                    |     GitHub       |
                    | Flask SourceCode |
                    +--------+---------+
                             |
                             |
                             ▼
                    +------------------+
                    |     Jenkins      |
                    |   CI/CD Pipeline |
                    +--------+---------+
                             |
         +-------------------+------------------+
         |                                      |
         ▼                                      ▼
+------------------+                +-------------------+
|   SonarQube      |                | Docker Build      |
| Code Analysis    |                | Docker Image      |
+------------------+                +---------+---------+
                                              |
                                              ▼
                                   +----------------------+
                                   |    Docker Hub        |
                                   | Image Repository     |
                                   +----------+-----------+
                                              |
                                              ▼
                                   +----------------------+
                                   | Docker Compose       |
                                   | Deployment           |
                                   +----------+-----------+
                                              |
                                              ▼
                                   +----------------------+
                                   | Flask Application    |
                                   +----------+-----------+
                                              |
                                              ▼
                                   +----------------------+
                                   | MySQL Database       |
                                   +----------------------+
```

# Technologies Used
## Technology	Purpose
| Technology           | Purpose                    |
| -------------------- | -------------------------- |
| Python               | Backend Development        |
| Flask                | Web Framework              |
| MySQL                | Database                   |
| Git                  | Version Control            |
| GitHub               | Source Code Repository     |
| Jenkins              | CI/CD Automation           |
| SonarQube            | Static Code Analysis       |
| Docker               | Containerization           |
| Docker Hub           | Image Repository           |
| Docker Compose       | Multi-container Deployment |
| Linux (Amazon Linux) | Deployment Environment     |

---
# Project Structure
Automated-CICD-Pipeline-for-Containerized-Flask-Application
│
├── app.py
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
├── Jenkinsfile
├── templates/
│   ├── login.html
│   ├── register.html
│   └── dashboard.html
├── static/
├── database/
│   └── init.sql
└── README.md

---
# CI/CD Pipeline Workflow

Developer
     │
     ▼
Push Code to GitHub
     │
     ▼
Jenkins Pipeline Trigger
     │
     ▼
Clone Repository
     │
     ▼
SonarQube Code Analysis
     │
     ▼
Quality Gate
     │
     ▼
Build Docker Image
     │
     ▼
Push Image to Docker Hub
     │
     ▼
Docker Compose Deployment
     │
     ▼
Flask Application Running

---

---

# 3. Step 1: AWS EC2 Instance Preparation

## Launch EC2 Instance

- Open AWS EC2 Console
- Launch Ubuntu 22.04 LTS Instance
- Select `t2.micro`
- Create Key Pair

---

## Configure Security Group

| Type | Port |
|---|---|
| SSH | 22 |
| HTTP | 80 |
| Flask App | 5000 |
| Jenkins | 8080 |
| SonarQube | 9000 |

---

## Connect to EC2

```bash
ssh -i key.pem ubuntu@<ec2-public-ip>
```

---

# 4. Step 2: Install Dependencies on EC2

## Update Packages

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Install Docker, Git, Compose

```bash
sudo apt install git docker.io docker-compose-v2 -y
```

---

## Enable Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

---

## Add User to Docker Group

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

# 5. Step 3: Jenkins Installation and Setup

## Install Java

```bash
sudo apt install openjdk-21-jdk -y
```

---

## Install Jenkins

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```

```bash
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

```bash
sudo apt update
sudo apt install jenkins -y
```

---

## Start Jenkins

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---

## Get Jenkins Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Open:

```text
http://<ec2-public-ip>:8080
```

---

## Give Docker Permission to Jenkins

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```
# Install SonarQube
---
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community

---
---

# 6. Step 4: GitHub Repository Configuration

Project files:

```text
app.py
Dockerfile
docker-compose.yml
Jenkinsfile
requirements.txt
templates/
static/
```

---

# Dockerfile

```dockerfile
FROM python:3.9

WORKDIR /app

COPY . /app

RUN pip install --no-cache-dir flask flask-mysqldb bcrypt mysqlclient

EXPOSE 5000

CMD ["python", "app.py"]
```

---

# docker-compose.yml

```yaml
version: '3.8'

services:

  flask-app:
    image: akshay9480/flask-auth-app:latest
    container_name: flask-auth-app

    ports:
      - "5000:5000"

    depends_on:
      - mysql-db

    environment:
      MYSQL_HOST: mysql-db
      MYSQL_USER: root
      MYSQL_PASSWORD: root123
      MYSQL_DB: authapp

    restart: always

  mysql-db:
    image: mysql:8

    container_name: mysql-db

    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: authapp

    ports:
      - "3306:3306"

    restart: always

    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

---

# Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "akshay9480/flask-auth-app"
        IMAGE_TAG = "latest"
        DOCKER_CREDS = "Dockerhub"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/rajakumarck28/Automated-CICD-Pipeline-for-Containerized-Flask-Application.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('SonarQube') {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=flask-auth-app \
                        -Dsonar.projectName=flask-auth-app \
                        -Dsonar.sources=. \
                        -Dsonar.python.version=3
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_CREDS) {
                        def image = docker.image("${IMAGE_NAME}:${IMAGE_TAG}")
                        image.push()
                    }
                }
            }
        }

        stage('Docker Compose Rebuild') {
            steps {
                sh '''
                docker-compose up -d --build
                '''
            }
        }
    }

    post {
        success {
            echo 'Application deployed successfully!'
        }

        failure {
            echo 'Pipeline Failed!'
        }
    }
}
```

---

# 7. Step 5: Jenkins Pipeline Creation and Execution

## Create Jenkins Pipeline

- New Item
- Pipeline
- Pipeline Script from SCM
- Git
- Add GitHub Repository URL

---

## Build Pipeline

Click:

```text
Build Now
```

---

## Verify Containers

```bash
docker ps
```

---

## Access Application

```text
http://<ec2-public-ip>:5000
```

---

# 8. Conclusion

The CI/CD pipeline is fully automated using Jenkins and Docker. Whenever code is pushed to GitHub, Jenkins automatically builds and deploys the updated Flask application on AWS EC2.

---

# 9. Infrastructure Diagram

```text
AWS EC2
│
├── Jenkins
├── Docker
│   ├── Flask Container
│   └── MySQL Container
```

---

# 10. Workflow Diagram

```text
Developer
   ↓
GitHub Repository
   ↓
Jenkins Pipeline
   ↓
Docker Build
   ↓
Docker Compose
   ↓
Flask + MySQL Containers
   ↓
AWS EC2 Deployment
```
<img width="1920" height="1080" alt="Screenshot (136)" src="https://github.com/user-attachments/assets/9f3ff503-65d9-4391-ac76-0117fc59958f" />
<img width="1920" height="1080" alt="Screenshot (134)" src="https://github.com/user-attachments/assets/b0b00866-bb6a-4090-9fae-b35062811ce2" />
<img width="1920" height="1080" alt="Screenshot (138)" src="https://github.com/user-attachments/assets/6904dbf3-df54-44b0-8943-f32d76b52f50" />
<img width="1920" height="1080" alt="Screenshot (139)" src="https://github.com/user-attachments/assets/2b5be0df-0316-47f8-85fd-9a9336bfca02" />
<img width="1920" height="1080" alt="Screenshot (140)" src="https://github.com/user-attachments/assets/0e1117d3-d18e-47f2-8686-d1080d407f5c" />
