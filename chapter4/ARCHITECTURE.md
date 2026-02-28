# Chapter 4 - Recipe Sharing App Architecture

## Overview
Chapter 4 implements a serverless recipe sharing application with authentication, featuring a React frontend, API Gateway, Lambda functions, and DynamoDB backend.

## Architecture Components

### 1. Frontend Layer
- **S3 Bucket**: Hosts the React application (static files)
  - Bucket Name: `frontend-chapter-4-[stack-id]`
  - Access Control: Private with CloudFront access only
  - Website Configuration: Index document set to `index.html`

- **CloudFront Distribution**: CDN for frontend delivery
  - Origin: S3 bucket via Origin Access Identity
  - Default Root Object: `index.html`
  - Custom Error Response: 403 errors redirect to `/` (SPA routing)
  - HTTP/2 enabled
  - IPv6 enabled
  - Price Class: PriceClass_100

### 2. Authentication Layer
- **AWS Cognito User Pool**: `chapter4-userpool`
  - Auto-verified attributes: Email
  - Password Policy: Minimum 6 characters (relaxed for demo)
  - MFA: Disabled
  - Account Recovery: Via verified email
  - Admin Create User Only: Enabled
  - Email Configuration: Uses Cognito default email service

- **Cognito User Pool Client**: 
  - Client Name: `my-user-pool-client`
  - Generate Secret: Disabled (for frontend use)

- **JWT Authorizer**: 
  - Type: JWT
  - Identity Source: `$request.header.Authorization`
  - Audience: User Pool Client ID
  - Issuer: Cognito User Pool endpoint

### 3. API Layer
- **HTTP API Gateway**: 
  - Protocol: HTTP (not HTTPS in this setup)
  - CORS Configuration: Allows all origins, methods (GET, POST, PUT, DELETE), and headers
  - Stage: `dev` with auto-deploy enabled

### 4. Lambda Functions

#### 4.1 Health Check Lambda
- **Function Name**: `healthcheck`
- **Runtime**: Python 3.9
- **Route**: `GET /health`
- **Authorization**: None
- **Purpose**: Service health verification
- **Response**: `{"message": "Service is healthy"}`

#### 4.2 Auth Test Lambda
- **Function Name**: `testauth`
- **Runtime**: Python 3.9
- **Route**: `GET /auth`
- **Authorization**: JWT (requires valid token)
- **Purpose**: Verify JWT authentication
- **Response**: `{"message": "You've passed the authentication token"}`

#### 4.3 Get Recipes Lambda
- **Function Name**: `get-recipes`
- **Runtime**: Python 3.9
- **Route**: `GET /recipes`
- **Authorization**: None
- **Purpose**: Retrieve all recipes from DynamoDB
- **Permissions**: DynamoDB Scan, Query, GetItem
- **Response**: List of Recipe objects with ingredients and steps

#### 4.4 Post Recipe Lambda
- **Function Name**: `post-recipe`
- **Runtime**: Python 3.9
- **Route**: `POST /recipes`
- **Authorization**: JWT (requires valid token)
- **Purpose**: Create new recipe in DynamoDB
- **Permissions**: DynamoDB PutItem
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

#### 4.5 Delete Recipe Lambda
- **Function Name**: `delete-recipe`
- **Runtime**: Python 3.9
- **Route**: `DELETE /recipes/{recipe_id}`
- **Authorization**: JWT (requires valid token)
- **Purpose**: Delete recipe from DynamoDB
- **Permissions**: DynamoDB DeleteItem
- **Path Parameter**: `recipe_id`

#### 4.6 Like Recipe Lambda
- **Function Name**: `like-recipe`
- **Runtime**: Python 3.9
- **Route**: `PUT /recipes/like/{recipe_id}`
- **Authorization**: None
- **Purpose**: Increment likes count for a recipe
- **Permissions**: DynamoDB UpdateItem
- **Path Parameter**: `recipe_id`
- **Update Expression**: `SET likes = likes + :val`

### 5. Data Layer
- **DynamoDB Table**: `recipes`
  - Partition Key: `id` (String)
  - Billing Mode: PAY_PER_REQUEST (on-demand)
  - Attributes:
    - `id`: Recipe unique identifier (UUID)
    - `title`: Recipe name
    - `ingredients`: List of ingredient objects
    - `steps`: List of step objects
    - `likes`: Like counter (integer)

### 6. IAM Roles & Permissions

#### Lambda Execution Roles
Each Lambda function has a dedicated IAM role with:
- **Base Permission**: `AWSLambdaBasicExecutionRole` (CloudWatch Logs)
- **Additional Permissions**:
  - **Read Role** (Get Recipes): DynamoDB Scan, Query, GetItem
  - **Create Role** (Post Recipe): DynamoDB PutItem
  - **Delete Role** (Delete Recipe): DynamoDB DeleteItem
  - **Like Role** (Like Recipe): DynamoDB UpdateItem

## API Routes Summary

| Method | Route | Auth | Function | Purpose |
|--------|-------|------|----------|---------|
| GET | `/health` | None | healthcheck | Service health check |
| GET | `/auth` | JWT | testauth | Verify authentication |
| GET | `/recipes` | None | get-recipes | List all recipes |
| POST | `/recipes` | JWT | post-recipe | Create new recipe |
| DELETE | `/recipes/{recipe_id}` | JWT | delete-recipe | Delete recipe |
| PUT | `/recipes/like/{recipe_id}` | None | like-recipe | Increment likes |

## Data Flow

### User Registration & Authentication
1. User registers via Cognito User Pool
2. Admin creates user with temporary password
3. User receives email with credentials
4. User logs in and receives JWT token

### Recipe Operations
1. **View Recipes**: User → CloudFront → S3 (React App) → API Gateway → Get Recipes Lambda → DynamoDB
2. **Create Recipe**: User (authenticated) → API Gateway (JWT verified) → Post Recipe Lambda → DynamoDB
3. **Delete Recipe**: User (authenticated) → API Gateway (JWT verified) → Delete Recipe Lambda → DynamoDB
4. **Like Recipe**: User → API Gateway → Like Recipe Lambda → DynamoDB (increment counter)

## Security Features
- **Frontend**: Private S3 bucket with CloudFront Origin Access Identity
- **API**: JWT authorization on protected routes (POST, DELETE)
- **Database**: Fine-grained IAM permissions per Lambda function
- **CORS**: Configured to allow frontend requests

## Deployment
- **Infrastructure as Code**: CloudFormation template (`ch4-application-template.yaml`)
- **Frontend**: React application built and deployed to S3
- **Backend**: Lambda functions deployed via CloudFormation
- **Configuration**: Parameters for API name, User Pool name, username, and email

## Outputs
- HTTP API Endpoint: `https://{api-id}.execute-api.{region}.amazonaws.com/dev`
- Cognito User Pool ID
- Cognito User Pool Client ID
- CloudFront Distribution URL
