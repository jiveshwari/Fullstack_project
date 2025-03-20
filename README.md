# Deploying a Full-Stack Application with AWS CodePipeline 

**Note:** This documentation is **not 100% accurate**. I have documented what we implemented in our project to give you a **basic idea** of how things work. Some steps might vary depending on your setup, but this should serve as a solid reference.  


## ⚡ Deploying a Node.js Express App to AWS with CodePipeline, CodeBuild, and ECS Fargate

## **1️ Overview**

This guide outlines how to deploy a **Node.js Express app** to AWS using **Docker**, **AWS CodePipeline**, **CodeBuild**, **ECR**, **ECS (Fargate)**, and **ALB (Application Load Balancer)**. The deployment process is fully automated using GitHub as the source.

---

## **2️ Architecture Overview**

- **CodePipeline** → Automates the CI/CD process.
- **CodeBuild** → Builds the Docker image and pushes it to **ECR**.
- **ECR (Elastic Container Registry)** → Stores the Docker images.
- **ECS (Fargate)** → Runs the containerized app.
- **ALB (Application Load Balancer)** → Manages traffic routing.
- **Auto Scaling** → Adjusts container instances based on load.
- **AWS Parameter Store / Secrets Manager** → Manages environment variables securely.
- **Amazon RDS (PostgreSQL / MySQL)** → Manages the database.

---

## **3️ Prerequisites**

1. **AWS Account**
2. **GitHub Repository** with your Node.js Express app
3. **Docker Installed**
4. **AWS CLI Installed & Configured** (`aws configure`)
5. **IAM Role with Required Permissions** (ECS, ECR, CodePipeline, CodeBuild, ALB, etc.)

---
