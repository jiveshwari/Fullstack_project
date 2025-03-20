# 🚀🚀 Deploying a Full-Stack Application with AWS CodePipeline 🚀🚀 


🚨 **📝 Note:** This documentation is **not 100% accurate**. I have documented what we implemented in our project to give you a **basic idea** of how things work. Some steps might vary depending on your setup, but this should serve as a solid reference.  


## ⚡ Deploying a Node.js Express App to AWS with CodePipeline, CodeBuild, and ECS Fargate

## **1️⃣ Overview**

This guide outlines how to deploy a **Node.js Express app** to AWS using **Docker**, **AWS CodePipeline**, **CodeBuild**, **ECR**, **ECS (Fargate)**, and **ALB (Application Load Balancer)**. The deployment process is fully automated using GitHub as the source.

---

## **2️⃣ Architecture Overview**

- **CodePipeline** → Automates the CI/CD process.
- **CodeBuild** → Builds the Docker image and pushes it to **ECR**.
- **ECR (Elastic Container Registry)** → Stores the Docker images.
- **ECS (Fargate)** → Runs the containerized app.
- **ALB (Application Load Balancer)** → Manages traffic routing.
- **Auto Scaling** → Adjusts container instances based on load.
- **AWS Parameter Store / Secrets Manager** → Manages environment variables securely.
- **Amazon RDS (PostgreSQL / MySQL)** → Manages the database.

---

## **3️⃣ Prerequisites**

1. **AWS Account**
2. **GitHub Repository** with your Node.js Express app
3. **Docker Installed**
4. **AWS CLI Installed & Configured** (`aws configure`)
5. **IAM Role with Required Permissions** (ECS, ECR, CodePipeline, CodeBuild, ALB, etc.)

---

## **4️⃣ Project Setup**

### **A. Create `Dockerfile`**
```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install --production
COPY . .

EXPOSE 3000
CMD ["node", "app.js"]
```

### **B. Create `buildspec.yml` for CodeBuild**
```yaml
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
  build:
    commands:
      - echo Building the Docker image...
      - docker build -t my-app .
      - docker tag my-app:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/my-app:latest
  post_build:
    commands:
      - echo Pushing the Docker image...
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/my-app:latest
      - echo Writing image details to imagedefinitions.json...
      - printf '[{"name":"my-app-container","imageUri":"%s"}]' $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/my-app:latest > imagedefinitions.json
artifacts:
  files:
    - imagedefinitions.json
```

---

## **5️⃣ AWS Setup**

### **A. Create an ECR Repository**
```sh
aws ecr create-repository --repository-name my-app --region us-east-1
```

### **B. Set Up CodePipeline**
1. Go to **AWS CodePipeline**
2. Click **Create Pipeline** → Choose **GitHub as Source**
3. Add **Build Stage** → Choose **AWS CodeBuild**
4. Create a **new CodeBuild project**:
   - Runtime: **Ubuntu**
   - Environment: **Managed Image**
   - Build Commands from `buildspec.yml`
5. **Deploy Stage** → Choose **ECS (Fargate)**

---

## **6️⃣ Deploy to Amazon ECS (Fargate)**

### **A. Create ECS Cluster**
```sh
aws ecs create-cluster --cluster-name my-cluster
```

### **B. Create a Task Definition**
Define the container settings:
```json
{
  "family": "my-app-task",
  "containerDefinitions": [
    {
      "name": "my-app-container",
      "image": "AWS_ACCOUNT_ID.dkr.ecr.AWS_REGION.amazonaws.com/my-app:latest",
      "memory": 512,
      "cpu": 256,
      "essential": true,
      "portMappings": [{ "containerPort": 3000, "hostPort": 3000 }]
    }
  ]
}
```

### **C. Create Fargate Service**
```sh
aws ecs create-service --cluster my-cluster --service-name my-app-service --task-definition my-app-task --desired-count 2 --launch-type FARGATE
```

---

## **7️⃣ Optimizations**

### **A. Add an Application Load Balancer (ALB)**
1. **Create an ALB**
2. **Attach a Target Group** → Forward requests to ECS tasks
3. **Update ECS Service** → Attach ALB for load balancing

### **B. Implement Auto-Scaling**
```sh
aws application-autoscaling register-scalable-target --service-namespace ecs --resource-id service/my-cluster/my-app-service --scalable-dimension ecs:service:DesiredCount --min-capacity 1 --max-capacity 5
```

### **C. Use AWS Secrets Manager for Environment Variables**
```sh
aws secretsmanager create-secret --name my-app-secret --secret-string '{"DATABASE_URL":"postgres://username:password@host:5432/dbname"}'
```
Retrieve it in your app using AWS SDK:
```js
const AWS = require("aws-sdk");
const secretsManager = new AWS.SecretsManager();
async function getSecret() {
  const secret = await secretsManager.getSecretValue({ SecretId: "my-app-secret" }).promise();
  return JSON.parse(secret.SecretString);
}
```

---

## **8️⃣ Summary**  

✅ Push code to GitHub → CodePipeline triggers  
✅ CodeBuild builds and pushes Docker image to ECR  
✅ ECS Fargate runs the container  
✅ ALB manages traffic & Auto-Scaling optimizes performance  
✅ Secrets Manager securely manages environment variables  
✅ **Done!** Your backend is now hosted on AWS! 🎉

---

## 📌 **You can also add this additional steps**

- Test & Monitor your setup using **AWS CloudWatch**
- Set up **logging & monitoring** for ECS containers


────────────────────────────────────────────────────────

🎯 **Next Section** 🎯  

────────────────────────────────────────────────────────

## ⚡ Deploying React/Angular on AWS S3 + CloudFront with CodePipeline & CodeBuild

## **1️⃣ Overview**

This guide covers deploying a **React** or **Angular** project to **AWS S3 (Static Website Hosting)** with **CloudFront (CDN)** and automating the process using **AWS CodePipeline + CodeBuild**.

---

## **2️⃣ Architecture Overview**  

- **CodePipeline** → Automates the CI/CD process, ensuring seamless deployments.  
- **CodeBuild** → Installs dependencies, builds the project, and deploys it to S3.  
- **S3 (Simple Storage Service)** → Hosts the static files (React/Angular build output).  
- **CloudFront (CDN)** → Caches and delivers content globally for faster load times.  
- **Route 53 (Optional)** → Manages custom domain and DNS routing.  
- **ACM (AWS Certificate Manager)** → Provides SSL certificates for secure HTTPS access.  
- **IAM Roles & Policies** → Restricts access and ensures security.  
- **AWS WAF (Optional)** → Protects against malicious traffic and attacks.  

---

## **3️⃣ Prerequisites**

- AWS Account
- A React or Angular project (`npm run build` should generate `dist/` or `build/` folder)
- GitHub repository (for CodePipeline integration)
- AWS CLI & IAM permissions for S3, CloudFront, CodeBuild, and CodePipeline

---

## **4️⃣ Create an S3 Bucket for Hosting**

1. Go to **AWS S3 Console** → **Create Bucket**
2. Name it (e.g., `my-frontend-app`)
3. Uncheck **Block Public Access** (since CloudFront will handle access control)
4. Enable **Static Website Hosting**
5. Note down the **Bucket Name**

---

## **5️⃣ Set Up CloudFront (CDN)**

1. Go to **AWS CloudFront Console** → **Create Distribution**
2. Select **Origin Domain** → Choose your S3 bucket
3. **Viewer Protocol Policy** → Redirect HTTP to HTTPS
4. **Cache Policy** → Use AWS Managed Caching
5. **Create Distribution** and note the **CloudFront Domain Name**

---

## **6️⃣ Automate Deployment with CodePipeline & CodeBuild**

### **Create an IAM Role for CodeBuild**
1. Go to **IAM Console** → **Roles** → **Create Role**
2. Select **AWS Service** → **CodeBuild**
3. Attach Policies:
   - `AmazonS3FullAccess`
   - `CloudFrontFullAccess`
   - `CodeBuildAdminAccess`
   - `CodePipelineFullAccess`
4. Name the role **CodeBuild-S3-Deploy** and save it.

---

## **7️⃣ Set Up AWS CodeBuild**

1. Go to **AWS CodeBuild** → **Create Build Project**
2. **Project Name:** `frontend-build`
3. **Source Provider:** GitHub (Connect your repo)
4. **Environment:**
   - OS: `Ubuntu`
   - Runtime: `Standard`
   - Image: `aws/codebuild/standard:latest`
   - Set **Privileged Mode: ON**
5. **Buildspec Configuration:** Choose "Insert Buildspec File"

#### **Add `buildspec.yml` in the root of your project**
```yaml
version: 0.2
phases:
  install:
    runtime-versions:
      nodejs: 18
  pre_build:
    commands:
      - npm install
  build:
    commands:
      - npm run build
  post_build:
    commands:
      - aws s3 sync ./build s3://my-frontend-app/ --delete
      - aws cloudfront create-invalidation --distribution-id YOUR_DISTRIBUTION_ID --paths "/*"
artifacts:
  files:
    - '**/*'
  discard-paths: no
```

---

## **8️⃣ Set Up AWS CodePipeline**

1. Go to **AWS CodePipeline** → **Create Pipeline**
2. **Source Stage:**
   - Select **GitHub** → Connect Repo
   - Branch: `main`
3. **Build Stage:**
   - Select **AWS CodeBuild**
   - Choose the `frontend-build` project
4. **Deploy Stage:**
   - Choose **S3** as deployment provider
   - Select the `my-frontend-app` bucket
5. **Create Pipeline** → It will trigger automatic deployment!

---

## **9️⃣ Summary**  

✅ Push code to GitHub → CodePipeline triggers  
✅ CodeBuild installs dependencies and builds the project  
✅ S3 hosts the static files (React/Angular build output)  
✅ CloudFront caches and delivers content globally  
✅ Route 53 (if using custom domain) manages DNS  
✅ ACM provides SSL for secure HTTPS access
✅ **Done!** Your frontend is now hosted on AWS! 🎉

---

## 📌 **Additional Steps (Optional) - Setting Up Subdomains for Different Environments**

When working with multiple environments (Development, QA, and Production), you can create subdomains under a single domain.

### Example Subdomain Setup:

| Environment   | Subdomain        | Example           |
|---------------|------------------|-------------------|
| Production    | Main Domain      | `www.mycode.com`  |
| Development   | Subdomain        | `dev.mycode.com`  |
| QA            | Subdomain        | `qa.mycode.com`   |

## Steps to Configure Subdomains

### 1. Purchase a Domain
You only need **one domain**, e.g., `mycode.com`, from a registrar like:
- GoDaddy
- AWS Route 53

### 2. Configure DNS Records
Go to your domain registrar’s **DNS settings** and add the following **A Records** or **CNAME Records**:

| Subdomain         | Record Type | Value (Example)                   |
|------------------|------------|-----------------------------------|
| `dev.mycode.com` | **A Record** or **CNAME** | IP Address or Load Balancer |
| `qa.mycode.com`  | **A Record** or **CNAME** | IP Address or Load Balancer |
| `www.mycode.com` | **A Record** | IP Address of Production Server |

#### Example (AWS Route 53)
1. Open **Route 53**.
2. Select your **hosted zone** (`mycode.com`).
3. Click **Create Record**.
4. Set:
   - **Record Name:** `dev`
   - **Record Type:** `A` (or `CNAME` if using a Load Balancer)
   - **Value:** IP Address of Dev Server
5. Repeat for `qa.mycode.com` and `www.mycode.com`.

### 3. Deploy Apps to Different Servers
- **Development (`dev.mycode.com`)** → Deploy to a Dev server.
- **QA (`qa.mycode.com`)** → Deploy to a QA server.
- **Production (`www.mycode.com`)** → Deploy to the live server.

### 4. Automate Deployment
Use **CI/CD Pipelines** (AWS CodePipeline, GitHub Actions, etc.) to deploy code to the correct subdomain based on the branch:
- `main` branch → Production (`www.mycode.com`)
- `develop` branch → Dev (`dev.mycode.com`)
- `qa` branch → QA (`qa.mycode.com`)

---

## 📌 **Additional Steps (Optional) - Custom Domain, Caching & Security**

### **1. Custom Domain with HTTPS (Route 53 + ACM)**
- Use **Amazon Route 53** to manage a custom domain.
- Create an **SSL Certificate** in **AWS Certificate Manager (ACM)**.
- Attach the SSL certificate to your **CloudFront distribution**.

### **2. Optimize CloudFront Caching**
- Configure CloudFront to cache static assets (`.js`, `.css`, `.png`, etc.).
- Use **Cache Invalidation** efficiently to reduce unnecessary refreshes.
- Set long `TTL` values for unchanged assets.

### **3. Security Enhancements**
- Restrict S3 bucket access using **IAM Policies** (allow access only from CloudFront).
- Enable **WAF (Web Application Firewall)** for additional protection.

---
