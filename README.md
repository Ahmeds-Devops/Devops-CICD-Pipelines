# ⚙️ DevOps CI/CD Pipelines

> A collection of real-world CI/CD pipeline configurations used in production environments.
> This repository contains Jenkins, Code Pipeline, Github Actions pipelines for deploying microservices, frontends, and backend services to AWS EKS and S3.

![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![ECR](https://img.shields.io/badge/AWS_ECR-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)

---

## 📁 Repository Structure

```
devops-cicd-pipelines/
├── jenkins/
│   ├── backend/
│   │   ├── Jenkinsfile-backend-eks-deploy        ← Backend microservice → EKS
│   │   └── Jenkinsfile-datadecoding-eks-deploy   ← Data decoding service → EKS
│   ├── frontend/
│   │   └── Jenkinsfile-ui-s3-deploy              ← React/Node UI → S3 + CloudFront
│   └── README.md
├── github-actions/
│   └── README.md                                 ← Coming soon
├── aws-codepipeline/
│   └── README.md                                 ← Coming soon
└── README.md
```

---

## 🚀 Pipelines Overview

### 1. 🖥️ Frontend UI Pipeline → S3 Deployment
**File:** `jenkins/frontend/Jenkinsfile-ui-s3-deploy`

Automates the build and deployment of a React/Node.js frontend application to **AWS S3**.

**Pipeline Stages:**
| Stage | Description |
|---|---|
| `Clone Repository` | Pulls latest code from GitHub using stored credentials |
| `Fetch env parameter` | Fetches environment variables securely from **AWS SSM Parameter Store** |
| `Install dependencies` | Runs `npm install` with caching for faster builds |
| `Build` | Compiles the frontend app for the target environment |
| `Remove old UI files` | Cleans the S3 bucket before deploying new build |
| `Upload latest UI files` | Syncs the new build artifacts to **AWS S3** |

**Tools Used:** Jenkins, AWS SSM, npm, AWS S3, AWS CLI

---

### 2. 🔧 Backend Microservice Pipeline → EKS Deployment
**File:** `jenkins/backend/Jenkinsfile-datadecoding-eks-deploy`

Automates Docker image build, push to **AWS ECR**, and deployment to **Amazon EKS** with rollback support.

**Pipeline Stages:**
| Stage | Description |
|---|---|
| `Clone Repository` | Pulls latest code from GitHub (main branch) |
| `Fetch Environment Parameters` | Fetches `.env`, `Dockerfile`, and K8s manifest from **AWS SSM Parameter Store** |
| `Build Docker Images` | Builds Docker image, tags with **commit hash** + `latest`, pushes to **AWS ECR** |
| `Deploy to EKS` | Assumes IAM role via **STS**, updates kubeconfig, deploys to EKS with auto-rollback |

**Key Features:**
- 🔐 **Secure secrets management** — all configs pulled from AWS SSM Parameter Store, nothing hardcoded
- 🏷️ **Git commit hash tagging** — every Docker image tagged with the exact commit for full traceability
- 🔁 **Auto rollback** — if deployment fails, automatically rolls back to the previous working version
- 🔑 **IAM Role assumption via STS** — Jenkins assumes a scoped EKS access role for least-privilege deployments
- ✅ **Rollout status check** — pipeline waits and verifies deployment health before marking success

**Tools Used:** Jenkins, AWS SSM, Docker, AWS ECR, AWS EKS, kubectl, AWS STS, Kubernetes

---

## 🏗️ Architecture Flow

```
Developer pushes code
        ↓
   Jenkins triggered
        ↓
   Clone Repository
        ↓
   Fetch configs from SSM Parameter Store
        ↓
   Build Docker Image
        ↓
   Push to AWS ECR (tagged with commit hash)
        ↓
   Assume IAM Role (STS)
        ↓
   Deploy to Amazon EKS
        ↓
   Rollout Status Check
        ↓
   ✅ Success  or  ❌ Auto Rollback
```

---

## 🔐 Security Practices Used

- ✅ **No hardcoded secrets** — all credentials and configs stored in AWS SSM Parameter Store
- ✅ **IAM Role assumption** — Jenkins uses `sts:AssumeRole` for scoped EKS access (least privilege)
- ✅ **Docker image immutability** — images tagged with Git commit hash for full audit trail
- ✅ **Jenkins credentials store** — GitHub tokens stored securely in Jenkins, not in pipeline code
- ✅ **ECR authentication** — uses `aws ecr get-login-password` for secure, token-based Docker login

---

## 🛠️ Prerequisites

To use these pipelines in your own environment you will need:

- Jenkins server with the following plugins: Git, Pipeline, AWS Steps
- AWS CLI configured on Jenkins agent
- `kubectl` installed on Jenkins agent
- `jq` installed on Jenkins agent
- AWS ECR repository created
- AWS EKS cluster running
- AWS SSM Parameter Store values configured
- Jenkins credentials configured: GitHub token, AWS IAM credentials

---

## 📌 Environment Variables Reference

| Variable | Description |
|---|---|
| `envName` | Target environment (dev/staging/prod) |
| `serviceName` | Name of the microservice being deployed |
| `parameterName` | SSM Parameter Store path for env config |
| `REGION` | AWS region (e.g., ap-south-1) |
| `clusterName` | EKS cluster name |
| `namespace` | Kubernetes namespace for deployment |
| `AccountId` | AWS account ID |
| `ECR_REPO_NAME` | ECR repository name |
| `EKS_ASSUME_ROLE_ARN` | IAM role ARN for EKS access via STS |

---

## 📚 Related Repositories

| Repo | Description |
|---|---|
| [terraform-aws-infrastructure](https://github.com/Ahmeds-Devops) | Terraform code to provision the AWS infra these pipelines deploy to |
| [kubernetes-manifests](https://github.com/Ahmeds-Devops) | Kubernetes YAML manifests used in EKS deployments |

---

## 👨‍💻 Author

**Ahmed Hussain Shaikh** — DevOps Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/ahmed-shaikh-devops/)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Ahmeds-Devops)

> ⭐ If you found this useful, please star the repo — it helps others find it too!
