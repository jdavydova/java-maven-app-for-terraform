# 🚀 Complete CI/CD Pipeline with Jenkins, Docker & Terraform

## 📌 Project Overview

This project demonstrates a complete DevOps pipeline that:

- Builds a Java Maven application
- Creates a Docker image
- Pushes the image to Docker Hub
- Provisions AWS infrastructure using Terraform
- Deploys the application to an EC2 instance using Docker Compose

---

## 🏗️ Architecture

```
Jenkins → Maven Build → Docker Build → Docker Hub
        → Terraform → AWS EC2
        → SSH → Docker Compose Deployment
```

---

## ⚙️ Technologies Used

- Jenkins (CI/CD)
- Maven (Build tool)
- Docker & Docker Hub
- Terraform (Infrastructure as Code)
- AWS (EC2, VPC, Networking)
- Docker Compose

---

## 🔄 Pipeline Stages

### 1️⃣ Checkout Code
Jenkins pulls:
- Application repo
- Shared library (jenkins-shared-library)

---

### 2️⃣ Build Application

```bash
mvn package
```

Produces:
```
target/java-maven-app-1.0-SNAPSHOT.jar
```

---

### 3️⃣ Build Docker Image

```bash
docker build -t juliadavydova/demo-app:java-maven-2.0 .
```

Base image:
```
amazoncorretto:17-alpine-jdk
```

---

### 4️⃣ Push to Docker Hub

```bash
docker login
docker push juliadavydova/demo-app:java-maven-2.0
```

---

### 5️⃣ Provision Infrastructure (Terraform)

```bash
terraform init
terraform apply --auto-approve
```

Creates:
- VPC
- Subnet
- Internet Gateway
- Route Table
- Security Group
- EC2 Instance

Example:
```
ec2-public_ip = "16.170.237.254"
```

⚠️ Changing `user_data` recreates EC2

---

### 6️⃣ Deploy Application to EC2

```bash
scp server-cmds.sh ec2-user@<EC2-IP>:/home/ec2-user
scp docker-compose.yaml ec2-user@<EC2-IP>:/home/ec2-user

ssh ec2-user@<EC2-IP> bash ./server-cmds.sh <IMAGE> <USER> <PASSWORD>
```

---

## 📦 Deployment Script

```bash
#!/usr/bin/env bash

export IMAGE=$1
export DOCKER_USER=$2
export DOCKER_PWD=$3

echo $DOCKER_PWD | docker login -u $DOCKER_USER --password-stdin

docker compose -f docker-compose.yaml up -d

echo "success"
```

---

## 🐳 Docker Compose

- Pulls application image
- Pulls PostgreSQL image
- Creates network
- Starts containers

Example:
```
Container java-maven-app Started
Container postgres Started
```

---

## 🖥️ EC2 Setup (Terraform user_data)

```bash
#!/bin/bash
sudo yum update -y
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user

sudo yum install -y docker-compose-plugin
```

---

## 🔐 Jenkins Credentials

| ID | Purpose |
|----|--------|
| github-credentials | GitHub access |
| docker-hub-repo | Docker login |
| jenkins-aws_access_key_id | AWS access |
| jenkins-aws_secret_access_key | AWS secret |
| ec2-user | SSH key |

---

## ⚠️ Important Notes

### Terraform
```
terraform output -raw ec2-public_ip
```

### Docker Compose Warning
Remove deprecated:
```
version:
```

### Jenkins Security
Avoid exposing secrets via Groovy interpolation.

---

## ✅ Final Result

```
Finished: SUCCESS
```

Application:
```
http://16.170.237.254:8080
```

---

## 🎯 Summary

Automates:
```
Build → Package → Containerize → Push → Provision → Deploy
```

---

## 📎 Repository

- App: https://github.com/jdavydova/java-maven-app-for-terraform
- Shared Library: https://github.com/jdavydova/jenkins-shared-library
