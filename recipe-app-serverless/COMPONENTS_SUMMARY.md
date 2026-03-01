# Chapter 4 - Components Summary

## Quick Reference

### Total Components: 16

#### Frontend (2)
- S3 Bucket
- CloudFront Distribution

#### Authentication (3)
- Cognito User Pool
- Cognito User Pool Client
- JWT Authorizer

#### API (1)
- HTTP API Gateway

#### Compute (6)
- healthcheck Lambda
- testauth Lambda
- get-recipes Lambda
- post-recipe Lambda
- delete-recipe Lambda
- like-recipe Lambda

#### Data (1)
- DynamoDB Table (recipes)

#### IAM (2)
- Lambda Execution Roles (6 total, grouped as 2 categories)
- DynamoDB Access Policies

---

## Component Checklist

### Frontend Layer
- [ ] S3 Bucket created with private access
- [ ] CloudFront distribution configured
- [ ] Origin Access Identity (OAI) set up
- [ ] Custom error responses configured
- [ ] React app deployed to S3

### Authentication Layer
- [ ] Cognito User Pool created
- [ ] Email verification enabled
- [ ] User Pool Client configured
- [ ] JWT Authorizer created
- [ ] Authorization header configured

### API Layer
- [ ] HTTP API Gateway created
- [ ] CORS configured
- [ ] Routes defined (6 total)
- [ ] Stages configured (dev)
- [ ] Auto-deploy enabled

### Lambda Functions
- [ ] healthcheck function deployed
- [ ] testauth function deployed
- [ ] get-recipes function deployed
- [ ] post-recipe function deployed
- [ ] delete-recipe function deployed
- [ ] like-recipe function deployed
- [ ] Lambda Powertools layer attached (post-recipe)

### Data Layer
- [ ] DynamoDB table created
- [ ] Partition key configured (id)
- [ ] On-demand billing enabled
- [ ] Attributes defined

### IAM & Security
- [ ] Lambda execution roles created
- [ ] DynamoDB permissions assigned
- [ ] CloudWatch Logs permissions granted
- [ ] Least privilege principle applied

---

## API Endpoints

```
GET  /health              → healthcheck Lambda
GET  /auth                → testauth Lambda (JWT required)
GET  /recipes             → get-recipes Lambda
POST /recipes             → post-recipe Lambda (JWT required)
DELETE /recipes/{id}      → delete-recipe Lambda (JWT required)
PUT  /recipes/like/{id}   → like-recipe Lambda
```

---

## Key Features

### Security
✓ JWT-based authentication
✓ Private S3 bucket with CloudFront
✓ Fine-grained IAM permissions
✓ HTTPS enforcement
✓ CORS configuration

### Scalability
✓ Serverless architecture
✓ Auto-scaling Lambda
✓ On-demand DynamoDB
✓ Global CloudFront CDN
✓ No server management

### Performance
✓ CloudFront caching
✓ Lambda concurrent execution
✓ DynamoDB on-demand
✓ API Gateway optimization
✓ SPA routing support

### Cost Efficiency
✓ Pay-per-use pricing
✓ Free tier eligible
✓ No idle costs
✓ On-demand DynamoDB
✓ CloudFront edge caching

---

## Deployment Artifacts

### Infrastructure
- CloudFormation template: `ch4-application-template.yaml`
- Parameters: API name, User Pool name, admin credentials

### Frontend
- React application source code
- Build output: `dist/` directory
- Deployment: S3 sync

### Backend
- Lambda function code (Python 3.9)
- Lambda Powertools layer
- IAM role definitions

---

## Monitoring & Observability

### CloudWatch Logs
- Lambda execution logs
- API Gateway access logs
- Error tracking

### CloudWatch Metrics
- Lambda invocations
- Lambda errors
- Lambda duration
- API Gateway requests
- DynamoDB consumed capacity

### Alarms (Optional)
- Lambda error rate > 5%
- API Gateway 5xx errors
- DynamoDB throttling

---

## Testing Endpoints

### Health Check
```bash
curl https://{api-endpoint}/health
```

### Get Recipes
```bash
curl https://{api-endpoint}/recipes
```

### Like Recipe
```bash
curl -X PUT https://{api-endpoint}/recipes/like/{recipe-id}
```

### Create Recipe (Requires JWT)
```bash
curl -X POST https://{api-endpoint}/recipes \
  -H "Authorization: Bearer {jwt-token}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Pasta",
    "ingredients": [{"id": 1, "description": "Pasta"}],
    "steps": [{"id": 1, "description": "Boil water"}],
    "likes": 0
  }'
```

---

## Resource Naming Convention

```
{component-type}-chapter-4-{stack-id}

Examples:
- frontend-chapter-4-abc123
- api-chapter-4-abc123
- recipes-table-chapter-4-abc123
```

---

## Environment Variables

### Frontend (.env)
```
VITE_API_ENDPOINT=https://{api-id}.execute-api.{region}.amazonaws.com/dev
VITE_COGNITO_REGION={region}
VITE_COGNITO_USER_POOL_ID={user-pool-id}
VITE_COGNITO_CLIENT_ID={client-id}
```

### Lambda (Environment Variables)
```
RECIPES_TABLE=recipes
AWS_REGION={region}
```

---

## Troubleshooting Guide

### Issue: 403 Forbidden on S3
**Solution**: Check CloudFront OAI permissions

### Issue: JWT Authorization Fails
**Solution**: Verify Cognito User Pool ID and Client ID match

### Issue: DynamoDB Item Not Found
**Solution**: Check recipe ID format (should be UUID)

### Issue: Lambda Timeout
**Solution**: Increase timeout or optimize DynamoDB queries

### Issue: CORS Errors
**Solution**: Verify API Gateway CORS configuration

---

## Next Steps

1. Deploy CloudFormation template
2. Create admin user in Cognito
3. Build and deploy React frontend
4. Test all API endpoints
5. Configure CloudWatch alarms
6. Set up CI/CD pipeline

---

## Related Documentation

- [ARCHITECTURE.md](./ARCHITECTURE.md) - Detailed architecture
- [COMPONENTS_LIST.md](./COMPONENTS_LIST.md) - Complete component list
- [COMPONENT_INTERACTIONS.md](./COMPONENT_INTERACTIONS.md) - Interaction details
- [ARCHITECTURE_DIAGRAM.txt](./ARCHITECTURE_DIAGRAM.txt) - ASCII diagram

