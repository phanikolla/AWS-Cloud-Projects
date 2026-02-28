# Chapter 4 - Documentation Index

## Quick Navigation

### 📋 Start Here
- **README.md** - Overview and quick start guide
- **ARCHITECTURE.md** - Detailed component descriptions

### 🏗️ Architecture Documentation
- **ARCHITECTURE_VISUAL.md** - ASCII diagrams and visual flows
- **COMPLETE_COMPONENT_LIST.md** - All 39 components listed with details

### 💻 Code
- **code/frontend/** - React TypeScript application
- **code/platform/ch4-application-template.yaml** - CloudFormation template

---

## Documentation Files

### README.md
**Purpose**: Overview and quick start  
**Contains**:
- Project overview
- Quick start guide
- Architecture summary
- API routes
- Technology stack
- Deployment outputs
- Troubleshooting

**Read this first!**

---

### ARCHITECTURE.md
**Purpose**: Detailed technical architecture  
**Contains**:
- Frontend layer (CloudFront, S3, OAI)
- Authentication layer (Cognito, JWT)
- API layer (HTTP API Gateway)
- Lambda functions (6 total)
- Data layer (DynamoDB)
- IAM roles and permissions
- API routes summary
- Data flow diagrams
- Security features
- Deployment information
- Outputs

**Read this for technical details**

---

### ARCHITECTURE_VISUAL.md
**Purpose**: Visual representation of architecture  
**Contains**:
- High-level architecture diagram
- Detailed component architecture (ASCII)
- Frontend layer diagram
- Authentication layer diagram
- API layer diagram
- Compute layer diagram
- Data layer diagram
- Security layer diagram
- Data flow diagrams
- Request flow examples
- Deployment architecture
- Key features

**Read this for visual understanding**

---

### COMPLETE_COMPONENT_LIST.md
**Purpose**: Complete inventory of all components  
**Contains**:
- All 39 components listed
- Frontend components (3)
- Authentication components (3)
- API components (7)
- Compute components (6)
- Data components (1)
- Security components (6)
- Integration components (12)
- Monitoring components (1)
- Technology stack
- Deployment outputs
- Key characteristics

**Read this for complete component details**

---

## Component Summary

### Total: 39 Components

| Category | Count |
|----------|-------|
| Frontend | 3 |
| Authentication | 3 |
| API | 7 |
| Compute | 6 |
| Data | 1 |
| Security | 6 |
| Integration | 12 |
| Monitoring | 1 |

---

## Architecture Layers

### 1. Frontend Layer (3 components)
- CloudFront CDN
- S3 Bucket
- Origin Access Identity

### 2. Authentication Layer (3 components)
- Cognito User Pool
- User Pool Client
- JWT Authorizer

### 3. API Layer (7 components)
- HTTP API Gateway
- 6 API Routes

### 4. Compute Layer (6 components)
- Health Check Lambda
- Auth Test Lambda
- Get Recipes Lambda
- Post Recipe Lambda
- Delete Recipe Lambda
- Like Recipe Lambda

### 5. Data Layer (1 component)
- DynamoDB Table

### 6. Security Layer (6 components)
- 6 IAM Roles (one per Lambda)

### 7. Integration Layer (12 components)
- 6 Lambda Permissions
- 6 API Integrations

### 8. Monitoring Layer (1 component)
- CloudWatch Logs

---

## API Routes (6 total)

| # | Method | Route | Auth | Lambda |
|---|--------|-------|------|--------|
| 1 | GET | `/health` | None | healthcheck |
| 2 | GET | `/auth` | JWT | testauth |
| 3 | GET | `/recipes` | None | get-recipes |
| 4 | POST | `/recipes` | JWT | post-recipe |
| 5 | DELETE | `/recipes/{id}` | JWT | delete-recipe |
| 6 | PUT | `/recipes/like/{id}` | None | like-recipe |

---

## Lambda Functions (6 total)

| # | Name | Runtime | Purpose |
|---|------|---------|---------|
| 1 | healthcheck | Python 3.9 | Service health |
| 2 | testauth | Python 3.9 | Auth verification |
| 3 | get-recipes | Python 3.9 | List recipes |
| 4 | post-recipe | Python 3.9 | Create recipe |
| 5 | delete-recipe | Python 3.9 | Delete recipe |
| 6 | like-recipe | Python 3.9 | Like recipe |

---

## IAM Roles (6 total)

| # | Role Name | Permissions |
|---|-----------|-------------|
| 1 | LambdaExecutionHCRole | CloudWatch Logs |
| 2 | LambdaExecutionAuthRole | CloudWatch Logs |
| 3 | LambdaExecutionReadRole | DynamoDB Read |
| 4 | LambdaExecutionCreateRecipeRole | DynamoDB Write |
| 5 | LambdaExecutionDeleteRecipeRole | DynamoDB Delete |
| 6 | LambdaExecutionLikeRecipeRole | DynamoDB Update |

---

## Key Features

✅ Serverless architecture  
✅ Auto-scaling  
✅ JWT authentication  
✅ Cost-effective  
✅ Global CDN  
✅ Reliable AWS services  
✅ Infrastructure as Code  

---

## Deployment

### CloudFormation Template
- **File**: `code/platform/ch4-application-template.yaml`
- **Resources**: 39 total
- **Parameters**: APIName, UserPoolName, Username, UserEmail

### Deployment Command
```bash
aws cloudformation create-stack \
  --stack-name chapter4-recipe-app \
  --template-body file://ch4-application-template.yaml \
  --parameters \
    ParameterKey=APIName,ParameterValue=chapter4-api \
    ParameterKey=UserPoolName,ParameterValue=chapter4-userpool \
    ParameterKey=Username,ParameterValue=testuser \
    ParameterKey=UserEmail,ParameterValue=test@example.com \
  --capabilities CAPABILITY_IAM
```

---

## Outputs

After deployment:
- **API Endpoint**: `https://{api-id}.execute-api.{region}.amazonaws.com/dev`
- **User Pool ID**: `{region}_{random}:userpool_id`
- **Client ID**: `{client_id}`
- **CloudFront URL**: `{distribution-id}.cloudfront.net`

---

## Technology Stack

### Frontend
- React 18 (TypeScript)
- Vite
- Tailwind CSS (optional)

### Backend
- AWS Lambda (Python 3.9)
- HTTP API Gateway
- DynamoDB
- Cognito

### Infrastructure
- CloudFormation
- AWS CLI

### Monitoring
- CloudWatch Logs

---

## Reading Guide

### For Beginners
1. Start with **README.md**
2. Review **ARCHITECTURE_VISUAL.md** for diagrams
3. Check **COMPLETE_COMPONENT_LIST.md** for details

### For Architects
1. Read **ARCHITECTURE.md** for technical details
2. Review **ARCHITECTURE_VISUAL.md** for flows
3. Check **COMPLETE_COMPONENT_LIST.md** for inventory

### For Developers
1. Check **README.md** for quick start
2. Review **code/platform/ch4-application-template.yaml**
3. Explore **code/frontend/** for React app

### For DevOps
1. Review **ARCHITECTURE.md** for deployment
2. Check **code/platform/ch4-application-template.yaml**
3. Use CloudFormation for deployment

---

## Quick Reference

### Component Count
- **Total**: 39 components
- **AWS Services**: 8 (CloudFront, S3, API Gateway, Lambda, DynamoDB, Cognito, IAM, CloudWatch)

### API Routes
- **Total**: 6 routes
- **Protected**: 3 (require JWT)
- **Public**: 3 (no auth)

### Lambda Functions
- **Total**: 6 functions
- **Runtime**: Python 3.9
- **Timeout**: 60 seconds each

### Database
- **Type**: DynamoDB
- **Billing**: On-demand
- **Partition Key**: id (String)

---

## Related Resources

- [AWS Lambda](https://docs.aws.amazon.com/lambda/)
- [API Gateway](https://docs.aws.amazon.com/apigateway/)
- [DynamoDB](https://docs.aws.amazon.com/dynamodb/)
- [Cognito](https://docs.aws.amazon.com/cognito/)
- [CloudFormation](https://docs.aws.amazon.com/cloudformation/)

---

## Support

For questions or issues:
1. Check the relevant documentation file
2. Review CloudWatch Logs
3. Verify IAM permissions
4. Check AWS service limits

---

**Chapter 4 - Complete Documentation** ✅

Last Updated: 2025-01-31
