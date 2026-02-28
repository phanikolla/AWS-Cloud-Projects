# Chapter 3 - Recipe Sharing Application

## Quick Links

- 📊 **Architecture Diagram**: `chapter3_architecture.png`
- 📋 **Architecture Documentation**: `ARCHITECTURE.md`
- 🎨 **ASCII Diagram**: `ARCHITECTURE_DIAGRAM.txt`

## Overview

This is a three-tier web application for sharing recipes, built with:
- **Frontend**: React + TypeScript (hosted on S3 + CloudFront)
- **Backend**: FastAPI (running on EC2)
- **Database**: DynamoDB (serverless NoSQL)

## Architecture at a Glance

```
Users
  ├─ HTTPS → CloudFront → S3 (Frontend)
  └─ HTTP → EC2 (Backend API) → DynamoDB
```

## Key Components

### Frontend
- React application with TypeScript
- Vite build tool
- Hosted on S3 (private bucket)
- Distributed via CloudFront CDN
- HTTPS enabled

### Backend
- FastAPI REST API
- Running on EC2 t3.micro instance
- Nginx reverse proxy on port 80
- CORS enabled for cross-origin requests
- IAM role for DynamoDB access

### Database
- DynamoDB table: `recipes`
- Primary key: `id` (String/UUID)
- Pay-per-request billing
- Stores recipe data with ingredients and steps

### Networking
- VPC: 10.0.0.0/16
- Public Subnet: 10.0.0.0/24
- Internet Gateway for external connectivity
- Security Group: HTTP (80) from anywhere

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check |
| GET | `/recipes` | Get all recipes |
| POST | `/recipes` | Create new recipe |
| DELETE | `/recipes/{id}` | Delete recipe |

## Deployment

### Prerequisites
- AWS Account
- CloudFormation access
- Git repository URL

### Deploy via CloudFormation
```bash
aws cloudformation create-stack \
  --stack-name chapter3-recipe-app \
  --template-body file://ch3-http.yaml \
  --parameters ParameterKey=GitRepoURL,ParameterValue=https://github.com/PacktPublishing/AWS-Cloud-Projects.git
```

### Outputs
After deployment, you'll receive:
- **CloudFront URL**: Access your frontend
- **EC2 Public DNS**: Access your backend API
- **CloudFront Distribution ID**: For cache management

## Security Features

✅ IAM roles with least privilege
✅ Security groups restricting traffic
✅ S3 bucket with public access blocked
✅ CloudFront Origin Access Identity
✅ CORS configuration
✅ DynamoDB encryption at rest

## Cost Optimization

- **EC2**: t3.micro (eligible for free tier)
- **DynamoDB**: Pay-per-request (scales automatically)
- **S3**: Minimal storage for static files
- **CloudFront**: Caches content globally

## Troubleshooting

### Frontend not loading
- Check CloudFront distribution status
- Verify S3 bucket policy
- Clear CloudFront cache

### API returning errors
- Check EC2 instance status
- Verify IAM role permissions
- Check DynamoDB table exists

### DynamoDB access denied
- Verify EC2 instance has correct IAM role
- Check DynamoDB table permissions
- Review CloudFormation outputs

## Files in This Directory

```
chapter3/
├── code/
│   ├── backend/
│   │   ├── main.py (FastAPI application)
│   │   └── requirements.txt
│   ├── frontend/
│   │   ├── src/ (React components)
│   │   ├── package.json
│   │   └── vite.config.ts
│   └── platform/
│       ├── ch3-http.yaml (CloudFormation template)
│       └── ch3-https-complete.yaml
├── ARCHITECTURE.md (Detailed documentation)
├── ARCHITECTURE_DIAGRAM.txt (ASCII diagram)
├── chapter3_architecture.png (Visual diagram)
└── README_ARCHITECTURE.md (This file)
```

## Next Steps

1. Review the architecture diagram
2. Read ARCHITECTURE.md for detailed information
3. Deploy using CloudFormation template
4. Test the application endpoints
5. Monitor costs in AWS Console

## Additional Resources

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [React Documentation](https://react.dev/)
- [DynamoDB Guide](https://docs.aws.amazon.com/dynamodb/)
- [CloudFormation Reference](https://docs.aws.amazon.com/cloudformation/)

---

**Last Updated**: January 31, 2026
**Status**: ✅ Complete with architecture diagrams
