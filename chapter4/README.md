# Chapter 4 - Recipe Sharing App (Serverless)

## Overview

Chapter 4 implements a **fully serverless recipe sharing application** with authentication, featuring:
- React TypeScript frontend
- HTTP API Gateway
- 6 Lambda functions
- Cognito authentication
- DynamoDB database
- CloudFront CDN

**Total Components**: 39 AWS resources

---

## Quick Start

### 1. Deploy Infrastructure

```bash
# Deploy using CloudFormation
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

### 2. Build Frontend

```bash
cd code/frontend
npm install
npm run build
```

### 3. Deploy Frontend

```bash
# Upload to S3
aws s3 sync dist/ s3://frontend-chapter-4-{stack-id}/

# Invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id {distribution-id} \
  --paths "/*"
```

### 4. Test API

```bash
# Get health
curl https://{api-id}.execute-api.{region}.amazonaws.com/dev/health

# Get recipes
curl https://{api-id}.execute-api.{region}.amazonaws.com/dev/recipes

# Create recipe (requires JWT token)
curl -X POST \
  -H "Authorization: Bearer {jwt-token}" \
  -H "Content-Type: application/json" \
  -d '{"title":"Pasta","ingredients":[...],"steps":[...],"likes":0}' \
  https://{api-id}.execute-api.{region}.amazonaws.com/dev/recipes
```

---

## Architecture

### High-Level View

```
Users
  ├─ Frontend (CloudFront + S3)
  ├─ Authentication (Cognito)
  └─ API (HTTP API Gateway)
      ├─ Health Check Lambda
      ├─ Auth Test Lambda
      ├─ Get Recipes Lambda → DynamoDB
      ├─ Post Recipe Lambda → DynamoDB
      ├─ Delete Recipe Lambda → DynamoDB
      └─ Like Recipe Lambda → DynamoDB
```

### Detailed Architecture

See `ARCHITECTURE_VISUAL.md` for complete ASCII diagrams

---

## Components

### Frontend (3 components)
- **CloudFront**: CDN for global distribution
- **S3 Bucket**: Hosts React application
- **Origin Access Identity**: Secures S3 access

### Authentication (3 components)
- **Cognito User Pool**: User management
- **User Pool Client**: Frontend authentication
- **JWT Authorizer**: API protection

### API (7 components)
- **HTTP API Gateway**: Request routing
- **6 API Routes**: Different operations

### Compute (6 components)
- **6 Lambda Functions**: Business logic
  - Health Check
  - Auth Test
  - Get Recipes
  - Post Recipe
  - Delete Recipe
  - Like Recipe

### Data (1 component)
- **DynamoDB Table**: Recipe storage

### Security (6 components)
- **6 IAM Roles**: Fine-grained permissions

### Integration (12 components)
- **6 Lambda Permissions**: API → Lambda
- **6 API Integrations**: Route → Lambda

### Monitoring (1 component)
- **CloudWatch Logs**: Function logging

**Total: 39 components**

---

## API Routes

| Method | Route | Auth | Purpose |
|--------|-------|------|---------|
| GET | `/health` | None | Health check |
| GET | `/auth` | JWT | Auth test |
| GET | `/recipes` | None | List recipes |
| POST | `/recipes` | JWT | Create recipe |
| DELETE | `/recipes/{id}` | JWT | Delete recipe |
| PUT | `/recipes/like/{id}` | None | Like recipe |

---

## Data Model

### Recipe Object

```json
{
  "id": "uuid",
  "title": "Recipe Name",
  "ingredients": [
    {
      "id": 1,
      "description": "Ingredient description"
    }
  ],
  "steps": [
    {
      "id": 1,
      "description": "Step description"
    }
  ],
  "likes": 0
}
```

---

## Documentation

### Architecture
- **ARCHITECTURE.md** - Detailed component descriptions
- **ARCHITECTURE_VISUAL.md** - ASCII diagrams and flows
- **COMPLETE_COMPONENT_LIST.md** - All 39 components listed

### Code
- **code/frontend/** - React TypeScript application
- **code/platform/ch4-application-template.yaml** - CloudFormation template

### Deployment
- **code/platform/ch4-application-template.yaml** - Infrastructure as Code

---

## Key Features

✅ **Serverless**: No servers to manage  
✅ **Scalable**: Auto-scales with demand  
✅ **Secure**: JWT authentication on protected routes  
✅ **Cost-Effective**: Pay only for what you use  
✅ **Global**: CloudFront CDN for fast delivery  
✅ **Reliable**: AWS managed services  
✅ **Maintainable**: Infrastructure as Code  

---

## Deployment Outputs

After deployment, you get:
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

## Security

### Authentication
- Cognito User Pool for user management
- JWT tokens for API authentication
- Protected routes require valid JWT

### Authorization
- Fine-grained IAM roles per Lambda function
- Each function has minimum required permissions
- DynamoDB access restricted by operation type

### Data Protection
- S3 bucket private with CloudFront access only
- HTTPS/TLS for all communications
- No sensitive data in logs

---

## Performance

### Frontend
- CloudFront CDN for global distribution
- S3 static hosting
- HTTP/2 enabled

### Backend
- Lambda auto-scaling
- DynamoDB on-demand billing
- No cold start issues for typical usage

### Monitoring
- CloudWatch Logs for debugging
- CloudWatch Metrics for monitoring

---

## Cost Estimation

### Monthly Costs (Typical Usage)
- **Lambda**: ~$0.20 (1M requests)
- **DynamoDB**: ~$1.25 (on-demand)
- **S3**: ~$0.50 (storage + requests)
- **CloudFront**: ~$0.50 (data transfer)
- **Cognito**: Free (up to 50K users)
- **Total**: ~$2.95/month

*Note: Actual costs depend on usage patterns*

---

## Troubleshooting

### API Returns 401 Unauthorized
- Check JWT token is valid
- Verify token is in Authorization header
- Check token hasn't expired

### Lambda Function Timeout
- Check function logs in CloudWatch
- Increase timeout if needed
- Check DynamoDB performance

### S3 Access Denied
- Verify CloudFront OAI is configured
- Check S3 bucket policy
- Verify files are uploaded

### DynamoDB Throttling
- Switch to provisioned capacity if needed
- Check for hot partitions
- Monitor consumed capacity

---

## Next Steps

1. **Deploy**: Use CloudFormation template
2. **Test**: Call API endpoints
3. **Monitor**: Check CloudWatch Logs
4. **Optimize**: Adjust based on usage
5. **Scale**: Add more features as needed

---

## Related Chapters

- **Chapter 3**: Recipe Sharing App (EC2-based)
- **Chapter 5**: Image Processing with Rekognition
- **Chapter 6**: CI/CD Pipeline
- **Chapter 7**: Chatbot with Bedrock
- **Chapter 8**: Data Pipeline

---

## Resources

- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [API Gateway Documentation](https://docs.aws.amazon.com/apigateway/)
- [DynamoDB Documentation](https://docs.aws.amazon.com/dynamodb/)
- [Cognito Documentation](https://docs.aws.amazon.com/cognito/)
- [CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)

---

## Support

For issues or questions:
1. Check CloudWatch Logs
2. Review architecture documentation
3. Check AWS service limits
4. Verify IAM permissions

---

**Chapter 4 - Serverless Recipe Sharing App** ✅
