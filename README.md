# ☁️ AWS Cloud Projects

[![AWS](https://img.shields.io/badge/AWS-Builder-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/)
[![CloudFormation](https://img.shields.io/badge/IaC-CloudFormation-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/cloudformation/)
[![Terraform](https://img.shields.io/badge/IaC-Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)](https://www.terraform.io/)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](./LICENSE)

## 📖 Introduction

A collection of **8 hands-on AWS projects** that progressively build real-world cloud skills — from hosting a static website to deploying AI-powered chatbots and business intelligence dashboards. Each project includes production-ready Infrastructure as Code (CloudFormation or Terraform), application source code, and architecture documentation.

### 🎯 What You'll Build

- Full-stack web applications with serverless backends
- CI/CD pipelines for automated deployments
- AI/ML-powered services using Rekognition, Lex, and Bedrock
- Data analytics dashboards with clickstream processing
- Secure, cost-optimized architectures following AWS best practices

---

## 📂 Project Directory

| # | Project | Difficulty | Tech Stack | Key Learning |
|:-:|:--------|:-----------|:-----------|:-------------|
| 1 | **[🖥️ EC2 Fundamentals](./ec2-fundamentals)** | 🟢 Beginner | EC2, CloudFormation, Terraform | Provisioning compute resources with Infrastructure as Code — both CloudFormation (YAML) and Terraform (HCL). |
| 2 | **[🌐 Professional CV Website](./static-cv-website)** | 🟢 Beginner | S3, CloudFront | Static website hosting with global CDN distribution, custom styling, and HTTPS. |
| 3 | **[🍳 Recipe Sharing App (Three-Tier)](./recipe-app-three-tier)** | 🟡 Intermediate | EC2, DynamoDB, S3, CloudFront, VPC, Nginx | Full-stack three-tier architecture — React frontend, FastAPI backend on EC2, DynamoDB database, with VPC networking and IAM security. |
| 4 | **[🍳 Recipe Sharing App (Serverless)](./recipe-app-serverless)** | 🟡 Intermediate | API Gateway, Lambda, DynamoDB, Cognito, S3, CloudFront | Serverless re-architecture of the Recipe app — API Gateway HTTP API, Lambda functions, Cognito authentication, and a React frontend. |
| 5 | **[📸 Photo Friendliness Analyzer](./photo-friendliness-analyzer)** | 🟡 Intermediate | Lambda, Rekognition, API Gateway, Terraform | AI-powered image analysis using Amazon Rekognition to evaluate photo friendliness for professional profiles. Deployed with Terraform. |
| 6 | **[🌍 Automated Content Translation Pipeline](./content-translation-pipeline)** | 🔴 Advanced | CodePipeline, CodeBuild, Lambda@Edge, CloudFront, S3, Terraform | End-to-end CI/CD pipeline that automatically translates website content across languages using Lambda@Edge at the CDN layer. |
| 7 | **[🤖 AI Q&A Chatbot](./ai-chatbot)** | 🔴 Advanced | Lex, Bedrock, Lambda, Cognito, API Gateway, DynamoDB | Conversational AI chatbot with Amazon Lex and Bedrock LLMs, user authentication via Cognito, and a React chat interface. |
| 8 | **[📊 Clickstream Analytics Dashboard](./clickstream-analytics)** | 🔴 Advanced | Kinesis, Glue, Athena, QuickSight, S3, CloudFormation | Business intelligence application for real-time website clickstream data ingestion, ETL processing, and interactive dashboards. |

---

## 🏗️ Architecture Highlights

Each project follows AWS Well-Architected Framework principles:

- **Security** — IAM least-privilege roles, security groups, S3 public access blocks, Cognito auth
- **Cost Optimization** — Serverless-first patterns, pay-per-request DynamoDB, t3.micro instances
- **Reliability** — CloudFront CDN for global availability, DynamoDB auto-scaling
- **Performance** — Edge caching, reverse proxies, optimized build pipelines
- **Operational Excellence** — Infrastructure as Code (CloudFormation + Terraform), CI/CD automation

---

## 🛠️ Prerequisites & Setup

### Requirements

| Tool | Version | Purpose |
|------|---------|---------|
| AWS CLI | v2+ | AWS resource management |
| AWS Account | Active | Services deployment |
| Node.js | v18+ | Frontend builds (React/Vite) |
| Python | 3.9+ | Backend development (FastAPI) |
| Terraform | 1.9+ | IaC for Chapters 1, 5, 6 |

### Quick Start

```bash
# Clone the repository
git clone https://github.com/phanikolla/AWS-Cloud-Projects.git
cd AWS-Cloud-Projects

# Example: Deploy Recipe Sharing App (Three-Tier)
aws cloudformation create-stack \
  --stack-name recipe-app-three-tier \
  --template-body file://recipe-app-three-tier/code/platform/ch3-http.yaml \
  --capabilities CAPABILITY_IAM \
  --parameters ParameterKey=GitRepoURL,ParameterValue=https://github.com/phanikolla/AWS-Cloud-Projects.git
```

---

## 📁 Repository Structure

```
AWS-Cloud-Projects/
├── ec2-fundamentals/                # EC2 provisioning (CloudFormation + Terraform)
├── static-cv-website/               # Static CV Website (S3 + CloudFront)
├── recipe-app-three-tier/           # Three-Tier Recipe App (EC2 + DynamoDB)
│   ├── code/
│   │   ├── backend/                     # FastAPI REST API
│   │   ├── frontend/                    # React + TypeScript + Vite
│   │   └── platform/                    # CloudFormation templates (HTTP & HTTPS)
│   └── ARCHITECTURE.md                  # Detailed architecture docs
├── recipe-app-serverless/           # Serverless Recipe App (API GW + Lambda + Cognito)
├── photo-friendliness-analyzer/     # Photo Analyzer (Rekognition + Terraform)
├── content-translation-pipeline/    # CI/CD Pipeline (CodePipeline + Lambda@Edge)
├── ai-chatbot/                      # AI Chatbot (Lex + Bedrock + Cognito)
├── clickstream-analytics/           # Clickstream Analytics (Kinesis + Glue + QuickSight)
└── README.md
```

---

## 💰 Cost Considerations

Most projects are designed to run within or near the **AWS Free Tier**:

| Project | Primary Cost Drivers | Estimated Monthly Cost |
|---------|---------------------|----------------------|
| EC2 Fundamentals / CV Website | EC2 t2.micro, S3, CloudFront | < $1 (Free Tier eligible) |
| Recipe App (Three-Tier) | EC2 t3.micro, DynamoDB, CloudFront | ~$12 (without Free Tier) |
| Recipe App (Serverless) | Lambda, API Gateway, DynamoDB | < $1 (pay-per-request) |
| Photo Analyzer | Lambda, Rekognition | < $1 (low usage) |
| Translation Pipeline | CodePipeline, Lambda@Edge | ~$2 |
| AI Chatbot | Lex, Bedrock, Lambda | Usage-based |
| Clickstream Analytics | Kinesis, Glue, QuickSight | Usage-based |

> ⚠️ **Remember**: Always delete your CloudFormation stacks and Terraform resources when done to avoid unexpected charges.

---

## 🤝 Contributing

Contributions are welcome! If you find a bug or have suggestions to improve the architecture or add optimizations:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -m 'Add improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

---

## 📬 Connect

- **LinkedIn**: [Phani Kumar Kolla](https://www.linkedin.com/in/phanikumarkolla/)
- **GitHub**: [@phanikolla](https://github.com/phanikolla)

---

<div align="center">

**☁️ Building on AWS — One Project at a Time ☁️**

*Crafted with expertise and passion by Phani Kolla*

</div>

---
