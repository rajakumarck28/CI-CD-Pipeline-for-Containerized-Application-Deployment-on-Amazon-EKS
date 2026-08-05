# DevOps Project Report: End-to-End DevOps CI/CD Pipeline for a Flask Application

# Project Overview

This project demonstrates how to implement a complete Continuous Integration and Continuous Deployment (CI/CD) pipeline for a Flask web application.

The pipeline automates the entire software delivery lifecycle—from source code management to deployment—ensuring faster, reliable, and consistent application releases.

The pipeline automatically:

- Clones the source code from GitHub
- Performs static code analysis using SonarQube
- Quality Gate validation
- Builds a Docker image
- Pushes the Docker image to Docker Hub
- Deploys the application using Docker Compose
- Connects the application with a MySQL database


# Technologies Used
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


# Project Structure
```text

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
```

# CI/CD Pipeline Workflow
```text

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
```
# Prerequisites

Install the following software before running the project:

- Python 3.x
- Git
- Docker
- Docker Compose
- Jenkins
- SonarQube Server
- Docker Hub Account

# Step 1: AWS EC2 Instance Preparation

## Launch EC2 Instance

- Open AWS EC2 Console
- Launch Ubuntu 22.04 LTS Instance
- Select `t2.micro`
- Create Key Pair


## Configure Security Group

| Type | Port |
|---|---|
| SSH | 22 |
| HTTP | 80 |
| Flask App | 5000 |
| Jenkins | 8080 |
| SonarQube | 9000 |


## Connect to EC2

```bash
ssh -i key.pem ubuntu@<ec2-public-ip>
```

---

# Step 2: Install Dependencies on EC2

## Update Packages

```bash
sudo apt update && sudo apt upgrade -y
```

## Install Docker, Git, Compose

```bash
sudo apt install git docker.io docker-compose-v2 -y
```

## Enable Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

## Add User to Docker Group

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

# Step 3: Jenkins Installation and Setup

## Install Java

```bash
sudo apt install openjdk-21-jdk -y
```

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

## Start Jenkins

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

## Get Jenkins Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Open:

```text
http://<ec2-public-ip>:8080
```

## Give Docker Permission to Jenkins

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```
# Install SonarQube
```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
```
---

Verify the installation:

- python --version
- git --version
- docker --version
- docker compose version
- java -version
- jenkins --version
  
# Step 4: Clone Repository

```bash
git clone https://github.com/rajakumarck28/Automated-CICD-Pipeline-for-Containerized-Flask-Application.git
```
```bash
cd Automated-CICD-Pipeline-for-Containerized-Flask-Application
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
# Run the Application Locally

## Docker

Build Docker Image
```bash
docker build -t akshay9480/flask-auth-app:latest .
```
Run Docker Container
```bash
docker run -d \
-p 5000:5000 \
--name flask-auth-app \
akshay9480/flask-auth-app:latest
```
Check Running Containers
```bash
docker ps
```

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
## Docker Compose
Docker Compose makes it easy to build and run the application with a single command.

Build and Start
```bash
docker compose up --build -d
```

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

# Step 5: Jenkins Pipeline Creation and Execution

## Create Jenkins Pipeline

- New Item
- Pipeline
- Pipeline Script from SCM
- Git
- Add GitHub Repository URL

## Build Pipeline

Click:

```text
Build Now
```

## Verify Containers

```bash
docker ps
```

## Access Application

```text
http://<ec2-public-ip>:5000
```

---

# Conclusion

This project demonstrates an end-to-end CI/CD pipeline for a containerized Flask application using GitHub, Jenkins, SonarQube, Docker, Docker Hub, Docker Compose, and MySQL. It automates code integration, quality analysis, Docker image creation, image publishing, and application deployment. The project highlights practical DevOps skills in CI/CD automation, containerization, code quality, and deployment, providing a scalable and reliable software delivery workflow.

---
# Future Enhancements
- Kubernetes Deployment
- AWS EKS Integration
- NGINX Reverse Proxy
- HTTPS with SSL/TLS
- GitHub Webhook Integration
- Automated Unit & Integration Testing
- Monitoring with Prometheus & Grafana

<img width="1920" height="1080" alt="Screenshot (55)" src="https://github.com/user-attachments/assets/2ec07bcf-8129-46c1-bd3a-11cc9be4571d" />
<img width="1920" height="1080" alt="Screenshot (57)" src="https://github.com/user-attachments/assets/42cc6cce-7184-49d2-9002-fcade77a7782" />
<img width="1920" height="1080" alt="Screenshot (56)" src="https://github.com/user-attachments/assets/060cddce-b2b1-4199-a7f3-6af51ee15aa8" />
<img width="1920" height="1080" alt="Screenshot (54)" src="https://github.com/user-attachments/assets/ee21fdde-5bb3-43c9-abf8-a16c76054bcb" />
<img width="1920" height="1080" alt="Screenshot (53)" src="https://github.com/user-attachments/assets/6bf31ae1-6dd8-426f-bbfc-93cae0664306" />






