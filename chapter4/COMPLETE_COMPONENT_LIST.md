# Chapter 4 - Complete Component List

## Overview
Chapter 4 implements a serverless recipe sharing application with authentication, featuring a React frontend, API Gateway, Lambda functions, and DynamoDB backend.

---

## 1. Frontend Components

### CloudFront Distribution
- **Type**: CDN (Content Delivery Network)
- **Purpose**: Distribute React application globally
- **Configuration**:
  - Origin: S3 bucket via Origin Access Identity
  - Default Root Object: index.html
  - Custom Error Response: 403 → / (for SPA routing)
  - HTTP/2 enabled
  - IPv6 enabled
  - Price Class: PriceClass_100
  - SSL/TLS: CloudFront default certificate

### S3 Bucket
- **Type**: Object Storage
- **Name**: `frontend-chapter-4-[stack-id]`
- **Purpose**: Host React application
- **Configuration**:
  - Access Control: Private
  - Public Access Block: All enabled
  - Website Configuration: Enabled
  - Versioning: Disabled
  - Content: React TypeScript application

### Origin Access Identity (OAI)
- **Type**: CloudFront Identity
- **Purpose**: Allow CloudFront to access S3 bucket
- **Configuration**:
  - Attached to S3 bucket policy
  - Restricts direct S3 access

---

## 2. Authentication Components

### Cognito User Pool
- **Type**: User Directory Service
- **Name**: `chapter4-userpool`
- **Purpose**: Manage user authentication
- **Configuration**:
  - Auto-verified attributes: Email
  - Password Policy: Minimum 6 characters (relaxed for demo)
  - MFA: Disabled
  - Account Recovery: Via verified email
  - Admin Create User: Enabled
  - Email Service: Cognito default
  - Schema: Email (required, mutable)

### Cognito User Pool Client
- **Type**: Application Client
- **Name**: `my-user-pool-client`
- **Purpose**: Frontend authentication
- **Configuration**:
  - Generate Secret: No (for frontend use)
  - Auth Flow: USER_PASSWORD_AUTH

### JWT Authorizer
- **Type**: API Gateway Authorizer
- **Purpose**: Validate JWT tokens on protected routes
- **Configuration**:
  - Type: JWT
  - Identity Source: `$request.header.Authorization`
  - Audience: User Pool Client ID
  - Issuer: Cognito IDP endpoint

---

## 3. API Components

### HTTP API Gateway
- **Type**: API Gateway (HTTP)
- **Purpose**: Route requests to Lambda functions
- **Configuration**:
  - Protocol: HTTP
  - Stage: `dev` (auto-deploy enabled)
  - CORS: All origins, methods (GET, POST, PUT, DELETE), headers
  - Endpoint: `https://{api-id}.execute-api.{region}.amazonaws.com/dev`

### API Routes (6 total)

#### Route 1: Health Check
- **Method**: GET
- **Path**: `/health`
- **Authorization**: None
- **Lambda**: healthcheck
- **Purpose**: Service health verification

#### Route 2: Auth Test
- **Method**: GET
- **Path**: `/auth`
- **Authorization**: JWT Required
- **Lambda**: testauth
- **Purpose**: Verify JWT authentication

#### Route 3: Get Recipes
- **Method**: GET
- **Path**: `/recipes`
- **Authorization**: None
- **Lambda**: get-recipes
- **Purpose**: Retrieve all recipes

#### Route 4: Post Recipe
- **Method**: POST
- **Path**: `/recipes`
- **Authorization**: JWT Required
- **Lambda**: post-recipe
- **Purpose**: Create new recipe

#### Route 5: Delete Recipe
- **Method**: DELETE
- **Path**: `/recipes/{recipe_id}`
- **Authorization**: JWT Required
- **Lambda**: delete-recipe
- **Purpose**: Delete recipe

#### Route 6: Like Recipe
- **Method**: PUT
- **Path**: `/recipes/like/{recipe_id}`
- **Authorization**: None
- **Lambda**: like-recipe
- **Purpose**: Increment likes counter

---

## 4. Compute Components (Lambda Functions)

### Lambda Function 1: Health Check
- **Name**: `healthcheck`
- **Runtime**: Python 3.9
- **Handler**: `index.lambda_handler`
- **Timeout**: 60 seconds
- **Memory**: 128 MB (default)
- **Purpose**: Service health verification
- **Response**: `{"message": "Service is healthy"}`

### Lambda Function 2: Auth Test
- **Name**: `testauth`
- **Runtime**: Python 3.9
- **Handler**: `index.lambda_handler`
- **Timeout**: 60 seconds
- **Memory**: 128 MB (default)
- **Purpose**: Verify JWT authentication
- **Response**: `{"message": "You've passed the authentication token"}`

### Lambda Function 3: Get Recipes
- **Name**: `get-recipes`
- **Runtime**: Python 3.9
- **Handler**: `index.lambda_handler`
- **Timeout**: 60 seconds
- **Memory**: 128 MB (default)
- **Purpose**: Retrieve all recipes from DynamoDB
- **Permissions**: DynamoDB Scan, Query, GetItem
- **Response**: List of Recipe objects

### Lambda Function 4: Post Recipe
- **Name**: `post-recipe`
- **Runtime**: Python 3.9
- **Handler**: `index.lambda_handler`
- **Timeout**: 60 seconds
- **Memory**: 128 MB (default)
- **Layers**: AWS Lambda Powertools Python V2
- **Purpose**: Create new recipe in DynamoDB
- **Permissions**: DynamoDB PutItem
- **Request Body**:
  ```json
  {
    "title": "string",
    "ingredients": [{"id": int, "description": "string"}],
    "steps": [{"id": int, "description": "string"}],
    "likes": int
  }
  ```

### Lambda Function 5: Delete Recipe
- **Name**: `delete-recipe`
- **Runtime**: Python 3.9
- **Handler**: `index.lambda_handler`
- **Timeout**: 60 seconds
- **Memory**: 128 MB (default)
- **Purpose**: Delete recipe from DynamoDB
- **Permissions**: DynamoDB DeleteItem
- **Path Parameter**: `recipe_id`

### Lambda Function 6: Like Recipe
- **Name**: `like-recipe`
- **Runtime**: Python 3.9
- **Handler**: `index.lambda_handler`
- **Timeout**: 60 seconds
- **Memory**: 128 MB (default)
- **Purpose**: Increment likes counter
- **Permissions**: DynamoDB UpdateItem
- **Path Parameter**: `recipe_id`
- **Update Expression**: `SET likes = likes + :val`

---

## 5. Data Components

### DynamoDB Table
- **Name**: `recipes`
- **Partition Key**: `id` (String)
- **Billing Mode**: PAY_PER_REQUEST (on-demand)
- **Attributes**:
  - `id`: Recipe unique identifier (UUID)
  - `title`: Recipe name (String)
  - `ingredients`: List of ingredient objects
    - `id`: Ingredient ID (Number)
    - `description`: Ingredient description (String)
  - `steps`: List of step objects
    - `id`: Step ID (Number)
    - `description`: Step description (String)
  - `likes`: Like counter (Number)

---

## 6. Security Components (IAM Roles)

### IAM Role 1: Lambda Execution Health Check Role
- **Name**: `LambdaExecutionHCRole`
- **Trust Policy**: Lambda service
- **Managed Policies**: AWSLambdaBasicExecutionRole
- **Custom Policies**: None
- **Purpose**: Execute health check Lambda

### IAM Role 2: Lambda Execution Auth Role
- **Name**: `LambdaExecutionAuthRole`
- **Trust Policy**: Lambda service
- **Managed Policies**: AWSLambdaBasicExecutionRole
- **Custom Policies**: None
- **Purpose**: Execute auth test Lambda

### IAM Role 3: Lambda Execution Read Role
- **Name**: `LambdaExecutionReadRole`
- **Trust Policy**: Lambda service
- **Managed Policies**: AWSLambdaBasicExecutionRole
- **Custom Policies**: DynamoDB Read Access
  - Actions: GetItem, Scan, Query
  - Resource: recipes table ARN
- **Purpose**: Execute get recipes Lambda

### IAM Role 4: Lambda Execution Create Recipe Role
- **Name**: `LambdaExecutionCreateRecipeRole`
- **Trust Policy**: Lambda service
- **Managed Policies**: AWSLambdaBasicExecutionRole
- **Custom Policies**: DynamoDB Write Access
  - Actions: PutItem
  - Resource: recipes table ARN
- **Purpose**: Execute post recipe Lambda

### IAM Role 5: Lambda Execution Delete Recipe Role
- **Name**: `LambdaExecutionDeleteRecipeRole`
- **Trust Policy**: Lambda service
- **Managed Policies**: AWSLambdaBasicExecutionRole
- **Custom Policies**: DynamoDB Delete Access
  - Actions: DeleteItem
  - Resource: recipes table ARN
- **Purpose**: Execute delete recipe Lambda

### IAM Role 6: Lambda Execution Like Recipe Role
- **Name**: `LambdaExecutionLikeRecipeRole`
- **Trust Policy**: Lambda service
- **Managed Policies**: AWSLambdaBasicExecutionRole
- **Custom Policies**: DynamoDB Update Access
  - Actions: UpdateItem
  - Resource: recipes table ARN
- **Purpose**: Execute like recipe Lambda

---

## 7. Integration Components

### Lambda Permissions (6 total)
- **Type**: Lambda resource-based policy
- **Purpose**: Allow API Gateway to invoke Lambda functions
- **Configuration**: One per Lambda function
  - Principal: apigateway.amazonaws.com
  - Action: lambda:InvokeFunction
  - Source ARN: API Gateway ARN

### API Gateway Integrations (6 total)
- **Type**: API Gateway integration
- **Purpose**: Connect routes to Lambda functions
- **Configuration**: One per route
  - Integration Type: AWS_PROXY
  - Integration URI: Lambda function ARN
  - Payload Format Version: 2.0

---

## 8. Monitoring Components

### CloudWatch Logs
- **Type**: Logging service
- **Purpose**: Capture Lambda function logs
- **Configuration**:
  - Log Group: `/aws/lambda/{function-name}`
  - Retention: Default (never expire)
  - Included via: AWSLambdaBasicExecutionRole

---

## Component Count Summary

| Category | Count | Details |
|----------|-------|---------|
| Frontend | 3 | CloudFront, S3, OAI |
| Authentication | 3 | User Pool, Client, JWT Authorizer |
| API | 7 | API Gateway, 6 Routes |
| Compute | 6 | Lambda Functions |
| Data | 1 | DynamoDB Table |
| Security | 6 | IAM Roles |
| Integration | 12 | 6 Permissions + 6 Integrations |
| Monitoring | 1 | CloudWatch Logs |
| **TOTAL** | **39** | **Complete serverless stack** |

---

## Technology Stack

### Frontend
- **Framework**: React (TypeScript)
- **Build Tool**: Vite
- **Hosting**: S3 + CloudFront

### Backend
- **API**: HTTP API Gateway
- **Compute**: AWS Lambda (Python 3.9)
- **Database**: DynamoDB
- **Authentication**: Cognito

### Infrastructure
- **IaC**: CloudFormation
- **Deployment**: CloudFormation Stack

### Monitoring
- **Logs**: CloudWatch Logs
- **Metrics**: CloudWatch Metrics (implicit)

---

## Deployment Outputs

After successful deployment, you receive:
1. **HTTP API Endpoint**: `https://{api-id}.execute-api.{region}.amazonaws.com/dev`
2. **Cognito User Pool ID**: `{region}_{random}:userpool_id`
3. **Cognito User Pool Client ID**: `{client_id}`
4. **CloudFront Distribution URL**: `{distribution-id}.cloudfront.net`

---

## Key Characteristics

✅ **Serverless**: No servers to manage  
✅ **Scalable**: Auto-scales with demand  
✅ **Secure**: JWT authentication on protected routes  
✅ **Cost-Effective**: Pay only for what you use  
✅ **Global**: CloudFront CDN for fast delivery  
✅ **Reliable**: AWS managed services  
✅ **Maintainable**: Infrastructure as Code (CloudFormation)  

---

## Next Steps

1. Review the architecture in `ARCHITECTURE.md`
2. Check the visual diagram in `ARCHITECTURE_VISUAL.md`
3. Deploy using CloudFormation template: `ch4-application-template.yaml`
4. Test the API endpoints
5. Monitor with CloudWatch Logs
