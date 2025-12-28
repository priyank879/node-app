# node-app
# 🚀 Node.js App Deployment on AWS ECS Fargate

This project demonstrates **automated deployment of a Node.js application** on **AWS ECS Fargate (serverless)** using **Docker**, **Amazon ECR**, **CloudWatch**, and **GitHub Actions CI/CD**.

## 🔧 Tech Stack
- Node.js
- Docker
- AWS ECS (Fargate)
- Amazon ECR
- CloudWatch Logs
- IAM Roles
- GitHub Actions (CI/CD)

## ⚙️ Architecture Flow
GitHub → GitHub Actions → Docker Build → ECR → ECS Fargate → CloudWatch

## 🔐 Security
- IAM Task Execution Role for ECS
- Secure image pull from ECR
- Logs integrated with CloudWatch

## 📦 Features
- Serverless container deployment
- Automated CI/CD pipeline
- Centralized logging
- Scalable & production-ready setup

## 📌 Use Case
DevOps practice | AWS portfolio | CI/CD automation demo
