# Chapter 4 - Component Interactions & Dependencies

## Component Interaction Matrix

### Frontend Components

#### S3 Bucket ↔ CloudFront
- **Direction**: Bidirectional
- **Type**: Origin relationship
- **Details**:
  - CloudFront retrieves static files from S3
  - Uses Origin Access Identity (OAI) for secure access
  - Caches responses based on TTL
  - Invalidates cache on deployment

#### CloudFront ↔ Users
- **Direction**: Bidirectional
- **Type**: Content delivery
- **Details**:
  - Users request React app through CloudFront
  - CloudFront serves cached content from edge locations
  - Redirects HTTP to HTTPS
  - Handles SPA routing (403 → /)

---

### Authentication Components

#### Cognito User Pool ↔ Users
- **Direction**: Bidirectional
- **Type**: Authentication service
- **Details**:
  - Users register/login via Cognito
  - Email verification required
  - Returns JWT token on successful authentication
  - Manages user credentials securely

#### Cognito ↔ API Gateway (JWT Authorizer)
- **Direction**: Unidirectional (API → Cognito)
- **Type**: Token validation
- **Details**:
  - API Gateway validates JWT tokens against Cognito
  - Checks token signature and expiration
  - Extracts user information from token claims
  - Denies requests with invalid/expired tokens

#### Cognito ↔ Protected Lambda Functions
- **Direction**: Unidirectional (Lambda ← API Gateway)
- **Type**: Authorization context
- **Details**:
  - testauth Lambda receives user context from JWT
  - post-recipe Lambda receives user context
  - delete-recipe Lambda receives user context
  - User ID available in event context

---

### API Layer Components

#### API Gateway ↔ Users
- **Direction**: Bidirectional
- **Type**: HTTP API endpoint
- **Details**:
  - Users send HTTP requests to API Gateway
  - API Gateway routes requests to appropriate Lambda
  - Returns JSON responses
  - Handles CORS for cross-origin requests

#### API Gateway ↔ Lambda Functions
- **Direction**: Bidirectional
- **Type**: Invocation & response
- **Details**:
  - API Gateway invokes Lambda synchronously
  - Passes HTTP request data as event
  - Lambda returns response object
  - API Gateway formats response as HTTP

#### API Gateway ↔ JWT Authorizer
- **Direction**: Unidirectional (API → Authorizer)
- **Type**: Authorization check
- **Details**:
  - API Gateway calls authorizer for protected routes
  - Authorizer validates JWT token
  - Returns authorization decision
  - Caches authorization result (5 minutes)

---

### Lambda Function Interactions

#### healthcheck Lambda
- **Inputs**: None
- **Outputs**: Health status JSON
- **Dependencies**: None
- **Interactions**: Standalone, no external calls

#### testauth Lambda
- **Inputs**: JWT token (from Authorization header)
- **Outputs**: Authentication confirmation
- **Dependencies**: Cognito (implicit via JWT validation)
- **Interactions**: Receives user context from API Gateway

#### get-recipes Lambda
- **Inputs**: None (optional query parameters)
- **Outputs**: List of recipes
- **Dependencies**: DynamoDB
- **Interactions**:
  - Calls DynamoDB Scan/Query operation
  - Receives recipe items
  - Formats and returns to API Gateway

#### post-recipe Lambda
- **Inputs**: Recipe object (title, ingredients, steps)
- **Outputs**: Created recipe with ID
- **Dependencies**: DynamoDB, Lambda Powertools
- **Interactions**:
  - Receives user context from JWT
  - Generates UUID for recipe ID
  - Calls DynamoDB PutItem
  - Returns created recipe

#### delete-recipe Lambda
- **Inputs**: recipe_id (path parameter)
- **Outputs**: Deletion confirmation
- **Dependencies**: DynamoDB
- **Interactions**:
  - Receives user context from JWT
  - Calls DynamoDB DeleteItem
  - Returns success/error response

#### like-recipe Lambda
- **Inputs**: recipe_id (path parameter)
- **Outputs**: Updated recipe with new like count
- **Dependencies**: DynamoDB
- **Interactions**:
  - Calls DynamoDB UpdateItem
  - Increments likes counter
  - Returns updated recipe

---

### Data Layer Components

#### DynamoDB Table ↔ Lambda Functions
- **Direction**: Bidirectional
- **Type**: Data operations
- **Details**:

| Lambda Function | Operations | Permissions |
|-----------------|-----------|-------------|
| get-recipes | Scan, Query, GetItem | Read-only |
| post-recipe | PutItem | Write |
| delete-recipe | DeleteItem | Delete |
| like-recipe | UpdateItem | Update |

#### DynamoDB ↔ IAM Roles
- **Direction**: Unidirectional (IAM → DynamoDB)
- **Type**: Access control
- **Details**:
  - Each Lambda has dedicated IAM role
  - Role defines allowed DynamoDB operations
  - Principle of least privilege applied
  - Resource-level permissions on recipes table

---

## Request/Response Flow Examples

### Example 1: View All Recipes (No Authentication)

```
1. User Browser
   ↓ GET /recipes
2. API Gateway
   ↓ Route to get-recipes Lambda
3. get-recipes Lambda
   ↓ DynamoDB Scan operation
4. DynamoDB
   ↓ Return all recipes
5. get-recipes Lambda
   ↓ Format response
6. API Gateway
   ↓ HTTP 200 + JSON
7. User Browser
   ↓ Display recipes
```

### Example 2: Create Recipe (With Authentication)

```
1. User Browser (Authenticated)
   ↓ POST /recipes + JWT token
2. API Gateway
   ↓ Call JWT Authorizer
3. JWT Authorizer
   ↓ Validate token with Cognito
4. Cognito
   ↓ Token valid, return user context
5. JWT Authorizer
   ↓ Return authorization decision
6. API Gateway
   ↓ Route to post-recipe Lambda
7. post-recipe Lambda
   ↓ Extract user context + recipe data
   ↓ Generate UUID
   ↓ DynamoDB PutItem
8. DynamoDB
   ↓ Store recipe
9. post-recipe Lambda
   ↓ Return created recipe
10. API Gateway
    ↓ HTTP 201 + JSON
11. User Browser
    ↓ Display confirmation
```

### Example 3: Delete Recipe (With Authentication)

```
1. User Browser (Authenticated)
   ↓ DELETE /recipes/{id} + JWT token
2. API Gateway
   ↓ Call JWT Authorizer
3. JWT Authorizer
   ↓ Validate token
4. Cognito
   ↓ Token valid
5. API Gateway
   ↓ Route to delete-recipe Lambda
6. delete-recipe Lambda
   ↓ Extract recipe_id from path
   ↓ DynamoDB DeleteItem
7. DynamoDB
   ↓ Delete recipe
8. delete-recipe Lambda
   ↓ Return success
9. API Gateway
   ↓ HTTP 200 + JSON
10. User Browser
    ↓ Remove from UI
```

### Example 4: Like Recipe (No Authentication)

```
1. User Browser
   ↓ PUT /recipes/like/{id}
2. API Gateway
   ↓ Route to like-recipe Lambda
3. like-recipe Lambda
   ↓ Extract recipe_id
   ↓ DynamoDB UpdateItem (likes + 1)
4. DynamoDB
   ↓ Increment counter
5. like-recipe Lambda
   ↓ Return updated recipe
6. API Gateway
   ↓ HTTP 200 + JSON
7. User Browser
   ↓ Update like count
```

---

## Dependency Graph

```
Users
├── CloudFront
│   └── S3 Bucket (React App)
├── API Gateway
│   ├── JWT Authorizer
│   │   └── Cognito User Pool
│   └── Lambda Functions
│       ├── healthcheck (no dependencies)
│       ├── testauth (Cognito context)
│       ├── get-recipes (DynamoDB)
│       ├── post-recipe (DynamoDB, Lambda Powertools)
│       ├── delete-recipe (DynamoDB)
│       └── like-recipe (DynamoDB)
└── Cognito User Pool
    └── Email Service
```

---

## Component Communication Protocols

### Frontend ↔ API Gateway
- **Protocol**: HTTPS
- **Format**: JSON
- **Authentication**: JWT in Authorization header
- **CORS**: Enabled for all origins

### API Gateway ↔ Lambda
- **Protocol**: AWS Lambda Invoke API
- **Format**: JSON event object
- **Synchronous**: Yes (HTTP API)
- **Timeout**: 30 seconds (default)

### Lambda ↔ DynamoDB
- **Protocol**: AWS SDK (boto3)
- **Format**: DynamoDB JSON
- **Asynchronous**: No (synchronous calls)
- **Retry**: Automatic with exponential backoff

### Lambda ↔ Cognito
- **Protocol**: AWS SDK (boto3)
- **Format**: JSON
- **Asynchronous**: No
- **Implicit**: Via JWT validation

---

## Error Handling & Fallbacks

### API Gateway Errors
- **4xx Errors**: Invalid request, authentication failure
- **5xx Errors**: Lambda execution failure
- **CORS Errors**: Preflight request failures

### Lambda Errors
- **Timeout**: 30 seconds default
- **Memory**: 128 MB default
- **Permissions**: IAM role insufficient permissions
- **DynamoDB**: Throttling, item not found

### DynamoDB Errors
- **Throttling**: On-demand mode handles automatically
- **Item Not Found**: Lambda returns 404
- **Validation**: Invalid item structure

---

## Scaling Considerations

### Horizontal Scaling
- **Lambda**: Auto-scales to concurrent execution limit
- **DynamoDB**: On-demand mode scales automatically
- **CloudFront**: Global edge locations

### Vertical Scaling
- **Lambda Memory**: Increase for CPU-intensive operations
- **DynamoDB**: Provisioned capacity (if switching from on-demand)
- **API Gateway**: No manual scaling needed

### Bottlenecks
- **Lambda Cold Starts**: ~1-2 seconds for Python 3.9
- **DynamoDB Throttling**: Unlikely with on-demand billing
- **CloudFront Cache**: Depends on TTL configuration

---

## Security Interactions

### Authentication Flow
1. User logs in via Cognito
2. Cognito returns JWT token
3. User includes JWT in API requests
4. API Gateway validates JWT with Cognito
5. Lambda receives authenticated user context

### Authorization Flow
1. API Gateway checks route authorization
2. Protected routes require valid JWT
3. JWT Authorizer validates token signature
4. Cognito confirms token validity
5. Request proceeds or is rejected

### Data Protection
- S3 bucket private with OAI
- DynamoDB encryption at rest
- HTTPS for all communications
- IAM roles enforce least privilege

