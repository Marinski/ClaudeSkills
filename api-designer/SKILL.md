---
name: api-designer
description: "Design and document RESTful and GraphQL APIs with OpenAPI/Swagger specifications, authentication patterns, versioning strategies, and best practices. Use for: (1) Creating API specifications, (2) Designing REST endpoints, (3) GraphQL schema design, (4) API authentication and authorization, (5) API versioning strategies, (6) Documentation generation"
---

# API Designer

## Overview

This skill provides comprehensive guidance for designing, documenting, and implementing modern APIs. It covers both REST and GraphQL paradigms, with emphasis on industry best practices, clear documentation, and maintainable architecture.

## Core Capabilities

### REST API Design
- Resource-oriented endpoint design
- HTTP method semantics (GET, POST, PUT, PATCH, DELETE)
- Status code usage and error handling
- Request/response payload design
- Pagination, filtering, and sorting strategies

### GraphQL API Design
- Schema definition and type system
- Query and mutation design
- Resolver patterns
- Fragment and directive usage
- Performance optimization (N+1 problem)

### API Documentation
- OpenAPI 3.0 specification generation
- Interactive documentation (Swagger UI)
- Authentication documentation
- Example requests and responses
- Code generation from specs

### Authentication & Authorization
- OAuth 2.0 flows (authorization code, client credentials, PKCE)
- JWT token design and validation
- API key management
- Role-based access control (RBAC)
- Rate limiting and throttling

### API Versioning
- URL versioning (/v1/, /v2/)
- Header-based versioning
- Semantic versioning for breaking changes
- Deprecation strategies
- Backward compatibility

## When to Use This Skill

Use this skill when:
- Designing a new API from scratch
- Documenting an existing API
- Refactoring API endpoints for clarity
- Implementing authentication/authorization
- Planning API versioning strategy
- Creating OpenAPI specifications
- Designing GraphQL schemas

## REST API Design Workflow

### Step 1: Identify Resources

Identify the core resources (nouns) your API will expose:

```
Resources: Users, Posts, Comments, Tags

Collections:
- GET    /users              (List all users)
- POST   /users              (Create new user)

Individual Resources:
- GET    /users/{id}         (Get specific user)
- PUT    /users/{id}         (Update user - full)
- PATCH  /users/{id}         (Update user - partial)
- DELETE /users/{id}         (Delete user)

Nested Resources:
- GET    /users/{id}/posts   (Get user's posts)
- POST   /users/{id}/posts   (Create post for user)
```

### Step 2: Design URL Structure

Follow RESTful naming conventions:

**Best Practices**:
- Use plural nouns for collections: `/users`, `/posts`
- Use hyphens for multi-word resources: `/blog-posts`
- Keep URLs lowercase
- Use nesting for relationships: `/users/{id}/posts`
- Limit nesting depth to 2 levels maximum
- Use query parameters for filtering: `/posts?status=published&author=123`

**Examples**:
```
✅ Good:
GET /users
GET /users/123
GET /users/123/posts
GET /posts?published=true&limit=10

❌ Bad:
GET /getUsers
GET /user/123
GET /users/123/posts/comments/likes (too deep)
GET /posts/published (filter should be query param)
```

### Step 3: Choose HTTP Methods

Map operations to standard HTTP methods:

**GET**: Retrieve resource(s)
- Safe and idempotent
- No request body
- Cacheable
- Example: `GET /users/123`

**POST**: Create new resource
- Not idempotent
- Request body contains new resource data
- Returns 201 Created with Location header
- Example: `POST /users` with user data

**PUT**: Replace entire resource
- Idempotent
- Request body contains complete resource
- Returns 200 OK or 204 No Content
- Example: `PUT /users/123` with full user object

**PATCH**: Partial update
- Typically idempotent
- Request body contains only fields to update
- Returns 200 OK with updated resource
- Example: `PATCH /users/123` with `{"email": "new@example.com"}`

**DELETE**: Remove resource
- Idempotent
- No request body typically
- Returns 204 No Content or 200 OK
- Example: `DELETE /users/123`

### Step 4: Design Request/Response Payloads

Structure JSON payloads consistently:

**Request Example**:
```json
POST /users
{
  "username": "johndoe",
  "email": "john@example.com",
  "profile": {
    "firstName": "John",
    "lastName": "Doe",
    "bio": "Software engineer"
  }
}
```

**Response Example**:
```json
{
  "id": "usr_1234567890",
  "username": "johndoe",
  "email": "john@example.com",
  "profile": {
    "firstName": "John",
    "lastName": "Doe",
    "bio": "Software engineer"
  },
  "createdAt": "2025-10-25T10:30:00Z",
  "updatedAt": "2025-10-25T10:30:00Z"
}
```

**Naming Conventions**:
- Use camelCase for JSON field names
- Use ISO 8601 for timestamps
- Use consistent ID formats (prefixed: `usr_`, `post_`)
- Include metadata (createdAt, updatedAt, version)

### Step 5: Implement Error Handling

Design comprehensive error responses:

**Error Response Format**:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters",
    "details": [
      {
        "field": "email",
        "message": "Email format is invalid"
      },
      {
        "field": "username",
        "message": "Username already exists"
      }
    ],
    "requestId": "req_abc123xyz",
    "timestamp": "2025-10-25T10:30:00Z"
  }
}
```

**HTTP Status Codes**:
- `200 OK`: Successful GET, PUT, PATCH
- `201 Created`: Successful POST
- `204 No Content`: Successful DELETE
- `400 Bad Request`: Invalid request data
- `401 Unauthorized`: Missing or invalid authentication
- `403 Forbidden`: Authenticated but not authorized
- `404 Not Found`: Resource doesn't exist
- `409 Conflict`: Resource conflict (e.g., duplicate)
- `422 Unprocessable Entity`: Validation errors
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server error
- `503 Service Unavailable`: Temporary downtime

### Step 6: Add Pagination and Filtering

Implement pagination for collection endpoints:

**Cursor-Based Pagination** (recommended for large datasets):
```
GET /posts?limit=20&cursor=eyJpZCI6MTIzfQ

Response:
{
  "data": [...],
  "pagination": {
    "nextCursor": "eyJpZCI6MTQzfQ",
    "prevCursor": "eyJpZCI6MTAzfQ",
    "hasMore": true
  }
}
```

**Offset-Based Pagination** (simpler, good for small datasets):
```
GET /posts?limit=20&offset=40&sort=-createdAt

Response:
{
  "data": [...],
  "pagination": {
    "total": 500,
    "limit": 20,
    "offset": 40,
    "hasMore": true
  }
}
```

**Filtering and Sorting**:
```
GET /posts?status=published&author=123&tags=tech,api&sort=-createdAt,title
```

## GraphQL API Design Workflow

### Step 1: Define Schema Types

Create type definitions for your domain:

```graphql
type User {
  id: ID!
  username: String!
  email: String!
  profile: Profile
  posts(limit: Int, offset: Int): [Post!]!
  createdAt: DateTime!
}

type Profile {
  firstName: String
  lastName: String
  bio: String
  avatarUrl: String
}

type Post {
  id: ID!
  title: String!
  content: String!
  published: Boolean!
  author: User!
  tags: [String!]!
  comments: [Comment!]!
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

### Step 2: Design Queries

Define read operations:

```graphql
type Query {
  # Get single resources
  user(id: ID!): User
  post(id: ID!): Post

  # Get collections with filtering
  users(
    limit: Int = 10
    offset: Int = 0
    search: String
  ): UserConnection!

  posts(
    limit: Int = 10
    offset: Int = 0
    published: Boolean
    authorId: ID
    tags: [String!]
  ): PostConnection!

  # Search
  searchPosts(query: String!): [Post!]!
}

type UserConnection {
  edges: [User!]!
  totalCount: Int!
  pageInfo: PageInfo!
}
```

### Step 3: Design Mutations

Define write operations:

```graphql
type Mutation {
  # User operations
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!
  deleteUser(id: ID!): DeleteUserPayload!

  # Post operations
  createPost(input: CreatePostInput!): CreatePostPayload!
  publishPost(id: ID!): PublishPostPayload!
  addComment(postId: ID!, input: CommentInput!): AddCommentPayload!
}

input CreateUserInput {
  username: String!
  email: String!
  password: String!
  profile: ProfileInput
}

type CreateUserPayload {
  user: User
  errors: [Error!]
}
```

See `examples/graphql_schema.graphql` for complete schema example.

## Authentication Patterns

### OAuth 2.0 Authorization Code Flow

**Use Case**: Web applications with backend

```
1. Client redirects user to authorization server
   GET /oauth/authorize?
     client_id=CLIENT_ID&
     redirect_uri=CALLBACK_URL&
     response_type=code&
     scope=read write&
     state=RANDOM_STATE

2. User authenticates and grants permission

3. Authorization server redirects back with code
   GET CALLBACK_URL?code=AUTH_CODE&state=RANDOM_STATE

4. Client exchanges code for token
   POST /oauth/token
   {
     "grant_type": "authorization_code",
     "code": "AUTH_CODE",
     "redirect_uri": "CALLBACK_URL",
     "client_id": "CLIENT_ID",
     "client_secret": "CLIENT_SECRET"
   }

5. Response contains access token
   {
     "access_token": "eyJhbGc...",
     "token_type": "Bearer",
     "expires_in": 3600,
     "refresh_token": "def50200...",
     "scope": "read write"
   }
```

### JWT Token Design

**Token Structure**:
```json
{
  "header": {
    "alg": "RS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "usr_1234567890",
    "iat": 1698336000,
    "exp": 1698339600,
    "scope": ["read:posts", "write:posts"],
    "roles": ["user", "editor"]
  },
  "signature": "..."
}
```

**Using JWT in Requests**:
```
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

### API Key Authentication

**Simple API Key**:
```
X-API-Key: sk_live_abcdef1234567890
```

**Best Practices**:
- Use different keys for different environments (dev, staging, prod)
- Allow multiple keys per account for rotation
- Implement key expiration
- Log key usage for security auditing
- Never expose keys in client-side code

See `references/rest_best_practices.md` for detailed authentication patterns.

## API Versioning Strategies

### URL Versioning (Recommended)

```
/v1/users
/v2/users
```

**Pros**:
- Clear and explicit
- Easy to route and cache
- Simple to understand
- Good for major breaking changes

**Cons**:
- Can lead to URL proliferation
- Requires maintaining multiple codebases

### Header Versioning

```
Accept: application/vnd.myapi.v2+json
API-Version: 2
```

**Pros**:
- Clean URLs
- Same endpoint, different versions
- Good for content negotiation

**Cons**:
- Less visible
- Harder to test in browser
- Can confuse caching

### When to Version

Create a new version when:
- Changing response structure
- Removing fields
- Changing field types
- Modifying authentication
- Breaking existing client contracts

Don't version for:
- Adding new optional fields
- Adding new endpoints
- Bug fixes
- Performance improvements

## OpenAPI Specification

### Basic Structure

```yaml
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
  description: API for managing users and posts

servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: https://staging-api.example.com/v1
    description: Staging server

paths:
  /users:
    get:
      summary: List users
      parameters:
        - name: limit
          in: query
          schema:
            type: integer
            default: 10
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'

components:
  schemas:
    User:
      type: object
      required:
        - username
        - email
      properties:
        id:
          type: string
        username:
          type: string
        email:
          type: string
          format: email
```

See `examples/openapi_spec.yaml` for complete specification example.

### Generating Documentation

Use the helper script to generate and validate OpenAPI specs:

```bash
# Generate OpenAPI spec from code
python scripts/api_helper.py generate --input api.py --output openapi.yaml

# Validate existing spec
python scripts/api_helper.py validate --spec openapi.yaml

# Generate documentation site
python scripts/api_helper.py docs --spec openapi.yaml --output docs/
```

## Best Practices

### Consistency
- Use consistent naming conventions across all endpoints
- Standardize error response format
- Apply same authentication pattern everywhere
- Use uniform timestamp format (ISO 8601)

### Security
- Always use HTTPS in production
- Validate all input data
- Implement rate limiting
- Use proper authentication for all endpoints
- Never expose sensitive data in URLs
- Implement CORS properly

### Performance
- Use pagination for large datasets
- Implement caching headers (ETag, Cache-Control)
- Support compression (gzip)
- Use cursor-based pagination for real-time data
- Implement field selection/sparse fieldsets

### Documentation
- Document all endpoints with OpenAPI
- Provide example requests and responses
- Document error codes and meanings
- Include authentication instructions
- Keep documentation in sync with code

### Maintainability
- Version your API appropriately
- Provide deprecation warnings before removing features
- Write integration tests for all endpoints
- Monitor API usage and errors
- Keep backwards compatibility when possible

## Common Patterns

### Health Check Endpoint
```
GET /health
Response: { "status": "ok", "timestamp": "2025-10-25T10:30:00Z" }
```

### Batch Operations
```
POST /users/batch
{
  "operations": [
    { "method": "POST", "path": "/users", "body": {...} },
    { "method": "PATCH", "path": "/users/123", "body": {...} }
  ]
}
```

### Webhook Events
```
POST /webhooks/configure
{
  "url": "https://your-app.com/webhook",
  "events": ["user.created", "post.published"],
  "secret": "webhook_secret_key"
}
```

## Quick Reference

### REST Endpoint Design Checklist
- [ ] Use plural nouns for collections
- [ ] Limit URL nesting to 2 levels
- [ ] Use appropriate HTTP methods
- [ ] Return correct status codes
- [ ] Implement consistent error format
- [ ] Add pagination for collections
- [ ] Include filtering and sorting
- [ ] Document with OpenAPI
- [ ] Implement authentication
- [ ] Add rate limiting

### GraphQL Schema Checklist
- [ ] Define clear type hierarchy
- [ ] Use nullable types appropriately
- [ ] Implement pagination (connections)
- [ ] Design mutations with input types
- [ ] Return errors in payload
- [ ] Document schema with descriptions
- [ ] Implement authentication/authorization
- [ ] Optimize for N+1 queries (DataLoader)

## Additional Resources

- See `references/rest_best_practices.md` for comprehensive REST API patterns
- See `examples/openapi_spec.yaml` for complete OpenAPI 3.0 specification
- See `examples/graphql_schema.graphql` for full GraphQL schema example
- See `scripts/api_helper.py` for API tooling utilities
