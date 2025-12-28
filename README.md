# node-app   

# 🚀 Automated Deployment of Node.js App on AWS ECS Fargate (ECR + CloudWatch + IAM)

This repository explains **step-by-step** how to automate deployment of a **Node.js application** on **AWS ECS Fargate**, using **ECR** for Docker images, **CloudWatch** for logs, and **proper IAM roles**.

This is written in a **beginner → intermediate DevOps friendly** way so you can explain it confidently on GitHub / interviews / LinkedIn.

---

## 🧱 Architecture Overview

**Flow:**

1. Developer pushes code to GitHub
2. GitHub Actions builds Docker image
3. Image pushed to **Amazon ECR**
4. ECS Fargate pulls image from ECR
5. Application runs inside ECS Service
6. Logs sent to **CloudWatch Logs**

---

## 📁 Repository Structure

```
nodejs-ecs-fargate/
│── app/
│   ├── index.js
│   ├── package.json
│   └── package-lock.json
│
│── Dockerfile
│── .dockerignore
│
│── ecs-task-def.json
│
│── .github/workflows/deploy.yml
│
│── README.md
```

---

## 🟢 Step 1: Create Node.js Application

### `app/index.js`

```js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Node.js App running on ECS Fargate 🚀');
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### `app/package.json`

```json
{
  "name": "ecs-node-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

---

## 🐳 Step 2: Dockerize the Application

### `Dockerfile`

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY app/package*.json ./
RUN npm install

COPY app/ .

EXPOSE 3000
CMD ["npm", "start"]
```

### `.dockerignore`

```
node_modules
npm-debug.log
```

---

## 📦 Step 3: Create Amazon ECR Repository

```bash
aws ecr create-repository \
 --repository-name nodejs-ecs-repo \
 --region ap-south-1
```

Login to ECR:

```bash
aws ecr get-login-password --region ap-south-1 \
| docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com
```

---

## 🧑‍💼 Step 4: IAM Roles (Very Important)

### 1️⃣ ECS Task Execution Role

Attach policies:

* `AmazonECSTaskExecutionRolePolicy`
* `CloudWatchLogsFullAccess`

Trust Policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "ecs-tasks.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }]
}
```

---

## 📄 Step 5: ECS Task Definition

### `ecs-task-def.json`

```json
{
  "family": "nodejs-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::<ACCOUNT_ID>:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "node-container",
      "image": "<ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/nodejs-ecs-repo:latest",
      "portMappings": [{
        "containerPort": 3000,
        "protocol": "tcp"
      }],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/nodejs-app",
          "awslogs-region": "ap-south-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

---

## 🌐 Step 6: Create ECS Cluster & Service

```bash
aws ecs create-cluster --cluster-name nodejs-cluster
```

Create Service (Attach ALB manually or via console):

* Launch type: **Fargate**
* VPC + Public Subnet
* Assign public IP
* Security group: allow port **3000**

---

## 🔄 Step 7: GitHub Actions – CI/CD Automation

### `.github/workflows/deploy.yml`

```yaml
name: Deploy to ECS Fargate

on:
  push:
    branches: ["main"]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v3

    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ap-south-1

    - name: Login to Amazon ECR
      uses: aws-actions/amazon-ecr-login@v2

    - name: Build and Push Docker Image
      run: |
        docker build -t nodejs-ecs-repo .
        docker tag nodejs-ecs-repo:latest <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/nodejs-ecs-repo:latest
        docker push <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/nodejs-ecs-repo:latest

    - name: Deploy to ECS
      uses: aws-actions/amazon-ecs-deploy-task-definition@v1
      with:
        task-definition: ecs-task-def.json
        service: nodejs-service
        cluster: nodejs-cluster
        wait-for-service-stability: true
```

---

## 📊 Step 8: CloudWatch Logs

Logs auto-created:

```
CloudWatch → Log groups → /ecs/nodejs-app
```

You can see:

* App logs
* Errors
* Container restarts

---

## ✅ Final Output

* Node.js app running on **ECS Fargate**
* Docker images stored in **ECR**
* CI/CD via **GitHub Actions**
* Logs in **CloudWatch**
* Secure IAM role-based access

---

## 📌 Use This Repo For

✔ DevOps Portfolio
✔ GitHub Showcase
✔ Interview Explanation
✔ LinkedIn Project Post

---

🔥 *If you want, I can next:*

* Add **Terraform**
* Add **ALB + HTTPS**
* Add **Auto Scaling**
* Convert this into **production-grade setup**
