# Chapter 3 - Recipe Sharing Application Architecture

## Overview
This is a three-tier web application for sharing recipes, built with a React frontend, FastAPI backend, and DynamoDB database.

## Architecture Components

### 1. **Frontend Layer**
- **Technology**: React + TypeScript
- **Hosting**: Amazon S3 (static website hosting)
- **CDN**: CloudFront Distribution
- **Features**:
  - Create, read, and delete recipes
  - Admin and user pages
  - Responsive UI with Vite build tool

### 2. **Backend Layer**
- **Technology**: FastAPI (Python)
- **Hosting**: EC2 Instance (t3.micro)
- **Web Server**: Nginx (reverse proxy)
- **Features**:
  - REST API endpoints for recipe management
  - CORS enabled for cross-origin requests
  - Health check endpoint
  - Endpoints:
    - `GET /health` - Health check
    - `GET /recipes` - Retrieve all recipes
    - `POST /recipes` - Create new recipe
    - `DELETE /recipes/{recipe_id}` - Delete recipe

### 3. **Data Layer**
- **Database**: Amazon DynamoDB
- **Table**: `recipes`
- **Primary Key**: `id` (String)
- **Billing Mode**: Pay-per-request
- **Data Structure**:
  ```
  {
    id: string (UUID),
    title: string,
    ingredients: [{ id: int, description: string }],
    steps: [{ id: int, description: string }]
  }
  ```

### 4. **Networking**
- **VPC**: 10.0.0.0/16
- **Public Subnet**: 10.0.0.0/24
- **Internet Gateway**: Enables internet connectivity
- **Security Group**: Allows HTTP (port 80) from anywhere
- **Route Table**: Routes traffic to Internet Gateway

### 5. **IAM & Security**
- **EC2 Instance Role**: Allows EC2 to access DynamoDB
- **Permissions**:
  - `dynamodb:PutItem` - Create recipes
  - `dynamodb:Scan` - Read recipes
  - `dynamodb:DeleteItem` - Delete recipes
- **S3 Bucket Policy**: CloudFront Origin Access Identity for secure S3 access
- **Public Access Block**: Enabled on S3 bucket

## Data Flow

```
User Browser
    ↓
    ├─→ HTTPS → CloudFront → S3 (Frontend)
    │
    └─→ HTTP → EC2 (Backend API)
              ↓
           DynamoDB (Recipes Data)
```

## Deployment

### Infrastructure as Code
- **Template**: CloudFormation YAML
- **Stack Name**: Chapter 3 Recipe Sharing App
- **Parameters**:
  - Instance Type (default: t3.micro)
  - Latest Ubuntu 22.04 AMI
  - Git Repository URL

### Deployment Steps
1. EC2 instance launches with user data script
2. Dependencies installed (Python, Nginx, Git)
3. Repository cloned and backend code deployed
4. FastAPI application starts on port 8000
5. Nginx configured as reverse proxy on port 80
6. S3 bucket created for frontend
7. CloudFront distribution created for CDN

## Key Features

✅ **Scalability**: DynamoDB auto-scales with pay-per-request billing
✅ **Performance**: CloudFront caches frontend assets globally
✅ **Security**: IAM roles, security groups, and S3 access controls
✅ **Availability**: Multi-AZ capable with CloudFront
✅ **Cost-Effective**: Uses t3.micro for compute, serverless database

## Outputs

After deployment, you'll receive:
- **CloudFront URL**: Access your frontend application
- **EC2 Public DNS**: Access your backend API
- **CloudFront Distribution ID**: For cache invalidation

## Technologies Used

| Component | Technology |
|-----------|-----------|
| Frontend | React, TypeScript, Vite |
| Backend | FastAPI, Python |
| Database | DynamoDB |
| Hosting | S3, EC2, CloudFront |
| Infrastructure | CloudFormation |
| Networking | VPC, IGW, Security Groups |
| IAM | Roles, Policies |
