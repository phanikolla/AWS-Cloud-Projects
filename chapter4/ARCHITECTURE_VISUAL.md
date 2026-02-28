# Chapter 4 - Recipe Sharing App - Architecture Diagram

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              USERS / CLIENTS                                │
└────────────────────────────────────┬────────────────────────────────────────┘
                                     │
                    ┌────────────────┼────────────────┐
                    │                │                │
                    ▼                ▼                ▼
        ┌──────────────────┐  ┌──────────────┐  ┌──────────────┐
        │   CloudFront     │  │   Cognito    │  │  HTTP API    │
        │      CDN         │  │  User Pool   │  │   Gateway    │
        └────────┬─────────┘  └──────┬───────┘  └──────┬───────┘
                 │                   │                 │
                 ▼                   ▼                 ▼
        ┌──────────────────┐  ┌──────────────┐  ┌──────────────┐
        │   S3 Bucket      │  │ JWT Auth     │  │ Lambda Funcs │
        │  (React App)     │  │ Authorizer   │  │   (6 total)  │
        └──────────────────┘  └──────────────┘  └──────┬───────┘
                                                        │
                                    ┌───────────────────┼───────────────────┐
                                    │                   │                   │
                                    ▼                   ▼                   ▼
                            ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
                            │  DynamoDB    │  │  IAM Roles   │  │  CloudWatch  │
                            │  Recipes     │  │  (6 roles)   │  │    Logs      │
                            │   Table      │  └──────────────┘  └──────────────┘
                            └──────────────┘
```

---

## Detailed Component Architecture

### 1. Frontend Layer

```
┌─────────────────────────────────────────────────────────────┐
│                    FRONTEND LAYER                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  CloudFront Distribution                             │  │
│  │  ├─ Origin: S3 Bucket (via OAI)                      │  │
│  │  ├─ Cache Behavior: HTTP/2                          │  │
│  │  ├─ Custom Error: 403 → /                           │  │
│  │  └─ SSL/TLS: CloudFront Default Certificate         │  │
│  └──────────────────────────────────────────────────────┘  │
│                         │                                   │
│                         ▼                                   │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  S3 Bucket (frontend-chapter-4-[stack-id])          │  │
│  │  ├─ Access: Private (CloudFront only)               │  │
│  │  ├─ Website Config: index.html                      │  │
│  │  ├─ Content: React Application (TypeScript)         │  │
│  │  └─ Versioning: Disabled                            │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2. Authentication Layer

```
┌─────────────────────────────────────────────────────────────┐
│                 AUTHENTICATION LAYER                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Cognito User Pool (chapter4-userpool)              │  │
│  │  ├─ Auto-verified: Email                            │  │
│  │  ├─ Password Policy: Min 6 chars (relaxed)          │  │
│  │  ├─ MFA: Disabled                                   │  │
│  │  ├─ Admin Create User: Enabled                      │  │
│  │  └─ Email Service: Cognito Default                 │  │
│  └──────────────────────────────────────────────────────┘  │
│                         │                                   │
│                         ▼                                   │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  User Pool Client (my-user-pool-client)             │  │
│  │  ├─ Generate Secret: No                             │  │
│  │  └─ Auth Flow: USER_PASSWORD_AUTH                   │  │
│  └──────────────────────────────────────────────────────┘  │
│                         │                                   │
│                         ▼                                   │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  JWT Authorizer                                      │  │
│  │  ├─ Type: JWT                                       │  │
│  │  ├─ Identity Source: Authorization header          │  │
│  │  ├─ Audience: User Pool Client ID                  │  │
│  │  └─ Issuer: Cognito IDP endpoint                   │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3. API Layer

```
┌─────────────────────────────────────────────────────────────┐
│                      API LAYER                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  HTTP API Gateway                                    │  │
│  │  ├─ Protocol: HTTP                                  │  │
│  │  ├─ Stage: dev (auto-deploy)                        │  │
│  │  ├─ CORS: All origins, methods, headers             │  │
│  │  └─ Endpoint: https://{api-id}.execute-api.{region} │  │
│  └──────────────────────────────────────────────────────┘  │
│                         │                                   │
│         ┌───────────────┼───────────────┐                   │
│         │               │               │                   │
│         ▼               ▼               ▼                   │
│    ┌─────────┐    ┌─────────┐    ┌─────────┐              │
│    │ Public  │    │ Public  │    │Protected│              │
│    │ Routes  │    │ Routes  │    │ Routes  │              │
│    └─────────┘    └─────────┘    └─────────┘              │
│         │               │               │                   │
│         ▼               ▼               ▼                   │
│    GET /health    GET /recipes    POST /recipes            │
│    GET /auth      PUT /like/{id}   DELETE /{id}            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 4. Compute Layer (Lambda Functions)

```
┌─────────────────────────────────────────────────────────────┐
│                    COMPUTE LAYER                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  1. Health Check Lambda (healthcheck)               │  │
│  │     Route: GET /health                              │  │
│  │     Auth: None                                      │  │
│  │     Response: {"message": "Service is healthy"}     │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  2. Auth Test Lambda (testauth)                     │  │
│  │     Route: GET /auth                                │  │
│  │     Auth: JWT Required                              │  │
│  │     Response: {"message": "Auth token verified"}    │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  3. Get Recipes Lambda (get-recipes)                │  │
│  │     Route: GET /recipes                             │  │
│  │     Auth: None                                      │  │
│  │     DB: DynamoDB Scan                               │  │
│  │     Response: List of Recipe objects                │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  4. Post Recipe Lambda (post-recipe)                │  │
│  │     Route: POST /recipes                            │  │
│  │     Auth: JWT Required                              │  │
│  │     DB: DynamoDB PutItem                            │  │
│  │     Layers: AWS Lambda Powertools Python V2         │  │
│  │     Response: {"message": "Recipe created"}         │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  5. Delete Recipe Lambda (delete-recipe)            │  │
│  │     Route: DELETE /recipes/{recipe_id}              │  │
│  │     Auth: JWT Required                              │  │
│  │     DB: DynamoDB DeleteItem                         │  │
│  │     Response: {"message": "Recipe deleted"}         │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  6. Like Recipe Lambda (like-recipe)                │  │
│  │     Route: PUT /recipes/like/{recipe_id}            │  │
│  │     Auth: None                                      │  │
│  │     DB: DynamoDB UpdateItem (increment likes)       │  │
│  │     Response: {"message": "Recipe liked"}           │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 5. Data Layer

```
┌─────────────────────────────────────────────────────────────┐
│                      DATA LAYER                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  DynamoDB Table (recipes)                            │  │
│  │  ├─ Partition Key: id (String)                       │  │
│  │  ├─ Billing Mode: PAY_PER_REQUEST (on-demand)       │  │
│  │  └─ Attributes:                                      │  │
│  │     ├─ id: Recipe UUID                              │  │
│  │     ├─ title: Recipe name                           │  │
│  │     ├─ ingredients: List of objects                 │  │
│  │     │  └─ {id: int, description: string}            │  │
│  │     ├─ steps: List of objects                       │  │
│  │     │  └─ {id: int, description: string}            │  │
│  │     └─ likes: Integer counter                       │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 6. Security Layer (IAM Roles)

```
┌─────────────────────────────────────────────────────────────┐
│                    SECURITY LAYER                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Lambda Execution Roles (6 total)                    │  │
│  │                                                      │  │
│  │  Base Permission (all):                             │  │
│  │  └─ AWSLambdaBasicExecutionRole (CloudWatch Logs)   │  │
│  │                                                      │  │
│  │  Additional Permissions:                            │  │
│  │  ├─ Health Check Role: None (basic only)            │  │
│  │  ├─ Auth Test Role: None (basic only)               │  │
│  │  ├─ Get Recipes Role: DynamoDB Scan, Query, GetItem │  │
│  │  ├─ Post Recipe Role: DynamoDB PutItem              │  │
│  │  ├─ Delete Recipe Role: DynamoDB DeleteItem         │  │
│  │  └─ Like Recipe Role: DynamoDB UpdateItem           │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Data Flow Diagrams

### User Registration & Authentication Flow

```
User
  │
  ├─→ Cognito User Pool (Register/Login)
  │   └─→ Receives JWT Token
  │
  └─→ API Gateway (with JWT in Authorization header)
      └─→ JWT Authorizer validates token
          └─→ If valid: Route to Lambda
              └─→ If invalid: 401 Unauthorized
```

### Recipe Operations Flow

```
GET /recipes (Public)
  User → API Gateway → Get Recipes Lambda → DynamoDB Scan → Return recipes

POST /recipes (Protected)
  User (JWT) → API Gateway → JWT Authorizer → Post Recipe Lambda → DynamoDB PutItem

DELETE /recipes/{id} (Protected)
  User (JWT) → API Gateway → JWT Authorizer → Delete Recipe Lambda → DynamoDB DeleteItem

PUT /recipes/like/{id} (Public)
  User → API Gateway → Like Recipe Lambda → DynamoDB UpdateItem (increment)
```

---

## API Routes Summary

| Method | Route | Auth | Lambda Function | DB Operation |
|--------|-------|------|-----------------|--------------|
| GET | `/health` | None | healthcheck | None |
| GET | `/auth` | JWT | testauth | None |
| GET | `/recipes` | None | get-recipes | Scan |
| POST | `/recipes` | JWT | post-recipe | PutItem |
| DELETE | `/recipes/{recipe_id}` | JWT | delete-recipe | DeleteItem |
| PUT | `/recipes/like/{recipe_id}` | None | like-recipe | UpdateItem |

---

## Component Interactions

### Request Flow Example: Create Recipe

```
1. User (authenticated)
   ↓
2. HTTP POST /recipes with JWT token
   ↓
3. API Gateway receives request
   ↓
4. JWT Authorizer validates token
   ├─ Valid? → Continue
   └─ Invalid? → 401 Unauthorized
   ↓
5. Route to Post Recipe Lambda
   ↓
6. Lambda assumes IAM role (LambdaExecutionCreateRecipeRole)
   ↓
7. Lambda executes with DynamoDB PutItem permission
   ↓
8. Lambda writes to DynamoDB recipes table
   ├─ id: UUID
   ├─ title: from request
   ├─ ingredients: from request
   ├─ steps: from request
   └─ likes: 0
   ↓
9. Lambda returns success response
   ↓
10. API Gateway returns 200 OK to user
```

---

## Deployment Architecture

```
CloudFormation Stack (ch4-application-template.yaml)
├─ Frontend Resources
│  ├─ S3 Bucket
│  ├─ CloudFront Distribution
│  └─ Origin Access Identity
├─ Authentication Resources
│  ├─ Cognito User Pool
│  ├─ User Pool Client
│  └─ JWT Authorizer
├─ API Resources
│  ├─ HTTP API Gateway
│  └─ API Stage (dev)
├─ Compute Resources
│  ├─ 6 Lambda Functions
│  └─ 6 IAM Roles
└─ Data Resources
   └─ DynamoDB Table
```

---

## Key Features

✅ **Serverless Architecture**: No servers to manage  
✅ **Scalable**: Auto-scales with demand  
✅ **Secure**: JWT authentication on protected routes  
✅ **Cost-Effective**: Pay only for what you use  
✅ **Global**: CloudFront CDN for fast delivery  
✅ **Reliable**: AWS managed services  

---

## Outputs

After deployment, you get:
- HTTP API Endpoint: `https://{api-id}.execute-api.{region}.amazonaws.com/dev`
- Cognito User Pool ID
- Cognito User Pool Client ID
- CloudFront Distribution URL
