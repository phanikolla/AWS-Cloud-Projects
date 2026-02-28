# Chapter 4 - Recipe Sharing App - Complete Components List

## Architecture Overview
A serverless recipe sharing application with authentication, featuring React frontend, API Gateway, Lambda functions, and DynamoDB backend.

---

## 1. FRONTEND LAYER

### S3 Bucket
- **Name**: `frontend-chapter-4-[stack-id]`
- **Purpose**: Hosts React application static files
- **Configuration**:
  - Private bucket with CloudFront access only
  - Website configuration with index.html as default
  - Versioning: Enabled
  - Public Access Block: All enabled

### CloudFront Distribution
- **Purpose**: CDN for frontend delivery
- **Configuration**:
  - Origin: S3 bucket via Origin Access Identity (OAI)
  - Default Root Object: `index.html`
  - Custom Error Response: 403 → `/` (SPA routing)
  - HTTP/2: Enabled
  - IPv6: Enabled
  - Price Class: PriceClass_100
  - Viewer Protocol Policy: Redirect HTTP to HTTPS

---

## 2. AUTHENTICATION LAYER

### AWS Cognito User Pool
- **Name**: `chapter4-userpool`
- **Configuration**:
  - Auto-verified attributes: Email
  - Password Policy: Minimum 6 characters
  - MFA: Disabled
  - Account Recovery: Via verified email
  - Admin Create User Only: Enabled
  - Email Configuration: Cognito default email service

### Cognito User Pool Client
- **Name**: `my-user-pool-client`
- **Configuration**:
  - Generate Secret: Disabled (for frontend use)
  - Allowed OAuth Flows: Not configured
  - Callback URLs: Frontend CloudFront URL

### JWT Authorizer
- **Type**: JWT
- **Identity Source**: `$request.header.Authorization`
- **Audience**: User Pool Client ID
- **Issuer**: Cognito User Pool endpoint
- **Protected Routes**: POST, DELETE, GET /auth

---

## 3. API LAYER

### HTTP API Gateway
- **Protocol**: HTTP
- **CORS Configuration**:
  - Allowed Origins: `*` (all)
  - Allowed Methods: GET, POST, PUT, DELETE, OPTIONS
  - Allowed Headers: `*` (all)
  - Max Age: 300 seconds
- **Stage**: `dev` with auto-deploy enabled
- **Endpoint**: `https://{api-id}.execute-api.{region}.amazonaws.com/dev`

---

## 4. LAMBDA FUNCTIONS LAYER

### 4.1 Health Check Lambda
- **Function Name**: `healthcheck`
- **Runtime**: Python 3.9
- **Route**: `GET /health`
- **Authorization**: None
- **Purpose**: Service health verification
- **Response**: `{"message": "Service is healthy"}`
- **Execution Role**: Basic Lambda execution role

### 4.2 Auth Test Lambda
- **Function Name**: `testauth`
- **Runtime**: Python 3.9
- **Route**: `GET /auth`
- **Authorization**: JWT (required)
- **Purpose**: Verify JWT authentication
- **Response**: `{"message": "You've passed the authentication token"}`
- **Execution Role**: Basic Lambda execution role

### 4.3 Get Recipes Lambda
- **Function Name**: `get-recipes`
- **Runtime**: Python 3.9
- **Route**: `GET /recipes`
- **Authorization**: None
- **Purpose**: Retrieve all recipes from DynamoDB
- **Permissions**: 
  - `dynamodb:Scan`
  - `dynamodb:Query`
  - `dynamodb:GetItem`
- **Response**: List of Recipe objects with ingredients and steps
- **Execution Role**: Lambda execution role + DynamoDB read permissions

### 4.4 Post Recipe Lambda
- **Function Name**: `post-recipe`
- **Runtime**: Python 3.9
- **Route**: `POST /recipes`
- **Authorization**: JWT (required)
- **Purpose**: Create new recipe in DynamoDB
- **Permissions**: `dynamodb:PutItem`
- **Layers**: AWS Lambda Powertools Python V2 (for parsing)
- **Request Body**:
  ```json
  {
    "title": "string",
    "ingredients": [{"id": int, "description": "string"}],
    "steps": [{"id": int, "description": "string"}],
    "likes": int
  }
  ```
- **Execution Role**: Lambda execution role + DynamoDB write permissions

### 4.5 Delete Recipe Lambda
- **Function Name**: `delete-recipe`
- **Runtime**: Python 3.9
- **Route**: `DELETE /recipes/{recipe_id}`
- **Authorization**: JWT (required)
- **Purpose**: Delete recipe from DynamoDB
- **Permissions**: `dynamodb:DeleteItem`
- **Path Parameter**: `recipe_id` (UUID)
- **Execution Role**: Lambda execution role + DynamoDB delete permissions

### 4.6 Like Recipe Lambda
- **Function Name**: `like-recipe`
- **Runtime**: Python 3.9
- **Route**: `PUT /recipes/like/{recipe_id}`
- **Authorization**: None
- **Purpose**: Increment likes count for a recipe
- **Permissions**: `dynamodb:UpdateItem`
- **Path Parameter**: `recipe_id` (UUID)
- **Update Expression**: `SET likes = likes + :val`
- **Execution Role**: Lambda execution role + DynamoDB update permissions

---

## 5. DATA LAYER

### DynamoDB Table
- **Table Name**: `recipes`
- **Partition Key**: `id` (String - UUID)
- **Billing Mode**: PAY_PER_REQUEST (on-demand)
- **Attributes**:
  - `id`: Recipe unique identifier (UUID) - Primary Key
  - `title`: Recipe name (String)
  - `ingredients`: List of ingredient objects (List)
    - Each ingredient: `{"id": int, "description": string}`
  - `steps`: List of step objects (List)
    - Each step: `{"id": int, "description": string}`
  - `likes`: Like counter (Number) - Default: 0
- **TTL**: Not configured
- **Streams**: Not configured
- **Point-in-time Recovery**: Not configured

---

## 6. IAM ROLES & PERMISSIONS

### Lambda Execution Roles

#### Base Role (All Lambdas)
- **Policy**: `AWSLambdaBasicExecutionRole`
- **Permissions**:
  - `logs:CreateLogGroup`
  - `logs:CreateLogStream`
  - `logs:PutLogEvents`

#### Get Recipes Lambda Role
- **Additional Permissions**:
  - `dynamodb:Scan` on `recipes` table
  - `dynamodb:Query` on `recipes` table
  - `dynamodb:GetItem` on `recipes` table

#### Post Recipe Lambda Role
- **Additional Permissions**:
  - `dynamodb:PutItem` on `recipes` table

#### Delete Recipe Lambda Role
- **Additional Permissions**:
  - `dynamodb:DeleteItem` on `recipes` table

#### Like Recipe Lambda Role
- **Additional Permissions**:
  - `dynamodb:UpdateItem` on `recipes` table

---

## 7. API ROUTES SUMMARY

| Method | Route | Auth | Function | Purpose |
|--------|-------|------|----------|---------|
| GET | `/health` | None | healthcheck | Service health check |
| GET | `/auth` | JWT | testauth | Verify authentication |
| GET | `/recipes` | None | get-recipes | List all recipes |
| POST | `/recipes` | JWT | post-recipe | Create new recipe |
| DELETE | `/recipes/{recipe_id}` | JWT | delete-recipe | Delete recipe |
| PUT | `/recipes/like/{recipe_id}` | None | like-recipe | Increment likes |

---

## 8. DATA FLOW DIAGRAMS

### User Registration & Authentication Flow
```
User → Cognito User Pool → Email Verification → JWT Token
```

### View Recipes Flow
```
User → CloudFront → S3 (React App) → API Gateway → Get Recipes Lambda → DynamoDB
```

### Create Recipe Flow
```
User (Authenticated) → API Gateway (JWT verified) → Post Recipe Lambda → DynamoDB
```

### Delete Recipe Flow
```
User (Authenticated) → API Gateway (JWT verified) → Delete Recipe Lambda → DynamoDB
```

### Like Recipe Flow
```
User → API Gateway → Like Recipe Lambda → DynamoDB (increment counter)
```

---

## 9. SECURITY FEATURES

### Frontend Security
- Private S3 bucket with CloudFront Origin Access Identity
- HTTPS enforcement via CloudFront
- SPA routing with custom error handling

### API Security
- JWT authorization on protected routes (POST, DELETE)
- CORS configured for frontend domain
- API Gateway request validation

### Database Security
- Fine-grained IAM permissions per Lambda function
- No direct database access from frontend
- Encryption at rest (default DynamoDB encryption)

### Authentication Security
- Cognito User Pool with email verification
- JWT token-based authorization
- Admin-only user creation

---

## 10. DEPLOYMENT CONFIGURATION

### Infrastructure as Code
- **Template**: `ch4-application-template.yaml` (CloudFormation)
- **Language**: YAML

### Frontend Deployment
- React application built and deployed to S3
- Build output: `dist/` directory
- Deployment method: AWS CLI S3 sync

### Backend Deployment
- Lambda functions deployed via CloudFormation
- Python runtime: 3.9
- Deployment package: ZIP file

### Configuration Parameters
- API name
- User Pool name
- Username for admin user
- Email for admin user
- Stack ID for unique resource naming

---

## 11. OUTPUTS

After deployment, the following outputs are available:

- **HTTP API Endpoint**: `https://{api-id}.execute-api.{region}.amazonaws.com/dev`
- **Cognito User Pool ID**: `{region}_{random}:{user-pool-id}`
- **Cognito User Pool Client ID**: `{client-id}`
- **CloudFront Distribution URL**: `https://{distribution-id}.cloudfront.net`
- **S3 Bucket Name**: `frontend-chapter-4-{stack-id}`

---

## 12. ENVIRONMENT VARIABLES

### Frontend Configuration
- `VITE_API_ENDPOINT`: HTTP API Gateway endpoint
- `VITE_COGNITO_REGION`: AWS region
- `VITE_COGNITO_USER_POOL_ID`: Cognito User Pool ID
- `VITE_COGNITO_CLIENT_ID`: Cognito Client ID

### Lambda Environment Variables
- `RECIPES_TABLE`: DynamoDB table name (`recipes`)
- `AWS_REGION`: AWS region

---

## 13. MONITORING & LOGGING

### CloudWatch Logs
- **Log Groups**: `/aws/lambda/{function-name}`
- **Retention**: 7 days (default)
- **Metrics**: Invocations, Errors, Duration, Throttles

### CloudWatch Alarms (Optional)
- Lambda error rate
- API Gateway 4xx/5xx errors
- DynamoDB throttling

---

## 14. COST CONSIDERATIONS

### Estimated Monthly Costs (Low Usage)
- **S3**: ~$0.50 (storage + requests)
- **CloudFront**: ~$1-5 (data transfer)
- **API Gateway**: ~$0.35 (1M requests free tier)
- **Lambda**: ~$0.20 (1M requests free tier)
- **DynamoDB**: ~$1.25 (on-demand pricing)
- **Cognito**: Free (up to 50K MAU)

**Total**: ~$3-8/month for low usage

---

## 15. SCALABILITY & PERFORMANCE

### Auto-Scaling
- **Lambda**: Automatic scaling (concurrent executions)
- **DynamoDB**: On-demand billing (auto-scales)
- **CloudFront**: Global edge locations

### Performance Optimization
- CloudFront caching for static assets
- Lambda provisioned concurrency (optional)
- DynamoDB DAX (optional caching layer)

---

## 16. DISASTER RECOVERY

### Backup Strategy
- **S3**: Versioning enabled
- **DynamoDB**: Point-in-time recovery (optional)
- **Code**: Git repository

### Recovery Time Objective (RTO)
- Frontend: < 5 minutes (CloudFront cache invalidation)
- API: < 1 minute (Lambda auto-recovery)
- Database: < 15 minutes (DynamoDB restore)

