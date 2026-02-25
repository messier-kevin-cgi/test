# API Documentation

## 📋 Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Base URL](#base-url)
- [Common Headers](#common-headers)
- [Response Format](#response-format)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)
- [Pagination](#pagination)
- [Endpoints](#endpoints)
- [Examples](#examples)
- [SDKs & Libraries](#sdks--libraries)

## 🎯 Overview

> ⚠️ **Missing Information**: API endpoints not yet implemented. This document serves as a template and will be updated as APIs are developed.

This document describes the RESTful API for the Test repository project.

### API Version

- **Current Version**: v1
- **Protocol**: HTTPS
- **Format**: JSON
- **Style**: RESTful

### API Design Principles

- Resource-based URLs
- Standard HTTP methods (GET, POST, PUT, PATCH, DELETE)
- Consistent response structure
- Comprehensive error messages
- Version control via URL path

## 🔐 Authentication

### Authentication Methods

```mermaid
graph LR
    A[Client] -->|1. Login Request| B[Auth Server]
    B -->|2. Validate Credentials| C[Database]
    C -->|3. Return User Data| B
    B -->|4. Generate JWT| B
    B -->|5. Return Token| A
    A -->|6. Store Token| A
    A -->|7. Request + Token| D[API Server]
    D -->|8. Validate Token| B
    B -->|9. Token Valid| D
    D -->|10. Return Data| A
    
    style B fill:#2196F3,color:#fff
    style D fill:#4CAF50,color:#fff
```

### JWT Authentication

#### Obtaining a Token

**Request:**
```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepassword123"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "rt_abc123...",
    "expiresIn": 3600,
    "user": {
      "id": "user_123",
      "email": "user@example.com",
      "name": "John Doe"
    }
  }
}
```

#### Using the Token

Include the JWT token in the `Authorization` header:

```http
GET /api/v1/resource
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### Refreshing a Token

**Request:**
```http
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refreshToken": "rt_abc123..."
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 3600
  }
}
```

## 🌐 Base URL

### Environments

| Environment | Base URL | Description |
|-------------|----------|-------------|
| Production | `https://api.test.example.com/v1` | Live production API |
| Staging | `https://api-staging.test.example.com/v1` | Pre-production testing |
| Development | `http://localhost:3000/api/v1` | Local development |

> ⚠️ **Missing Information**: Actual domain names to be configured.

## 📝 Common Headers

### Request Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
Accept: application/json
X-Request-ID: {unique-request-id}
```

### Response Headers

```http
Content-Type: application/json
X-Request-ID: {unique-request-id}
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640000000
```

## 📤 Response Format

### Success Response

```json
{
  "success": true,
  "data": {
    "id": "123",
    "name": "Example Resource",
    "createdAt": "2024-01-01T00:00:00Z"
  },
  "metadata": {
    "requestId": "req_abc123",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}
```

### List Response (with pagination)

```json
{
  "success": true,
  "data": [
    {
      "id": "1",
      "name": "Item 1"
    },
    {
      "id": "2",
      "name": "Item 2"
    }
  ],
  "pagination": {
    "page": 1,
    "perPage": 20,
    "total": 100,
    "totalPages": 5,
    "hasNext": true,
    "hasPrev": false
  },
  "metadata": {
    "requestId": "req_abc123",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}
```

## ❌ Error Handling

### Error Response Format

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Email is required"
      },
      {
        "field": "password",
        "message": "Password must be at least 8 characters"
      }
    ]
  },
  "metadata": {
    "requestId": "req_abc123",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}
```

### HTTP Status Codes

| Code | Status | Description |
|------|--------|-------------|
| 200 | OK | Request successful |
| 201 | Created | Resource created successfully |
| 204 | No Content | Request successful, no content to return |
| 400 | Bad Request | Invalid request data |
| 401 | Unauthorized | Authentication required or failed |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Resource conflict (e.g., duplicate) |
| 422 | Unprocessable Entity | Validation error |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |
| 503 | Service Unavailable | Service temporarily unavailable |

### Error Codes

| Code | Description |
|------|-------------|
| `VALIDATION_ERROR` | Input validation failed |
| `AUTHENTICATION_ERROR` | Authentication failed |
| `AUTHORIZATION_ERROR` | Insufficient permissions |
| `NOT_FOUND` | Resource not found |
| `CONFLICT` | Resource conflict |
| `RATE_LIMIT_EXCEEDED` | Too many requests |
| `INTERNAL_ERROR` | Internal server error |
| `SERVICE_UNAVAILABLE` | Service temporarily unavailable |

## ⏱️ Rate Limiting

### Rate Limit Rules

| Endpoint Type | Limit | Window |
|---------------|-------|--------|
| Authentication | 5 requests | 15 minutes |
| Read Operations | 1000 requests | 1 hour |
| Write Operations | 100 requests | 1 hour |
| Default | 500 requests | 1 hour |

### Rate Limit Headers

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640000000
```

### Rate Limit Exceeded Response

```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Try again in 3600 seconds.",
    "retryAfter": 3600
  }
}
```

## 📄 Pagination

### Query Parameters

```http
GET /api/v1/resources?page=1&perPage=20&sortBy=createdAt&sortOrder=desc
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number (starts at 1) |
| `perPage` | integer | 20 | Items per page (max 100) |
| `sortBy` | string | `createdAt` | Field to sort by |
| `sortOrder` | string | `desc` | Sort order (`asc` or `desc`) |

### Pagination Response

```json
{
  "pagination": {
    "page": 1,
    "perPage": 20,
    "total": 100,
    "totalPages": 5,
    "hasNext": true,
    "hasPrev": false,
    "links": {
      "first": "/api/v1/resources?page=1&perPage=20",
      "prev": null,
      "next": "/api/v1/resources?page=2&perPage=20",
      "last": "/api/v1/resources?page=5&perPage=20"
    }
  }
}
```

## 🔌 Endpoints

> ⚠️ **Missing Information**: Specific endpoints to be defined based on application requirements.

### Authentication Endpoints

#### Login

```http
POST /api/v1/auth/login
```

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securepassword123"
}
```

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "token": "jwt_token_here",
    "user": { "id": "123", "email": "user@example.com" }
  }
}
```

#### Register

```http
POST /api/v1/auth/register
```

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securepassword123",
  "name": "John Doe"
}
```

**Response:** `201 Created`

#### Logout

```http
POST /api/v1/auth/logout
Authorization: Bearer {token}
```

**Response:** `204 No Content`

### User Endpoints

#### Get Current User

```http
GET /api/v1/users/me
Authorization: Bearer {token}
```

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "id": "user_123",
    "email": "user@example.com",
    "name": "John Doe",
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
}
```

#### Update User Profile

```http
PATCH /api/v1/users/me
Authorization: Bearer {token}
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Jane Doe",
  "bio": "Software Developer"
}
```

**Response:** `200 OK`

#### Delete Account

```http
DELETE /api/v1/users/me
Authorization: Bearer {token}
```

**Response:** `204 No Content`

### Resource Endpoints

> Template for CRUD operations on a resource

#### List Resources

```http
GET /api/v1/resources
Authorization: Bearer {token}
```

**Query Parameters:**
- `page` (optional): Page number
- `perPage` (optional): Items per page
- `filter[field]` (optional): Filter by field
- `search` (optional): Search query

**Response:** `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "id": "res_123",
      "name": "Resource 1",
      "description": "Description",
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "perPage": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

#### Get Resource by ID

```http
GET /api/v1/resources/:id
Authorization: Bearer {token}
```

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "id": "res_123",
    "name": "Resource 1",
    "description": "Description",
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
}
```

#### Create Resource

```http
POST /api/v1/resources
Authorization: Bearer {token}
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "New Resource",
  "description": "Resource description"
}
```

**Response:** `201 Created`
```json
{
  "success": true,
  "data": {
    "id": "res_124",
    "name": "New Resource",
    "description": "Resource description",
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

#### Update Resource

```http
PUT /api/v1/resources/:id
Authorization: Bearer {token}
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Updated Resource",
  "description": "Updated description"
}
```

**Response:** `200 OK`

#### Partial Update Resource

```http
PATCH /api/v1/resources/:id
Authorization: Bearer {token}
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Partially Updated"
}
```

**Response:** `200 OK`

#### Delete Resource

```http
DELETE /api/v1/resources/:id
Authorization: Bearer {token}
```

**Response:** `204 No Content`

## 💡 Examples

### Example 1: Complete User Registration Flow

```bash
# 1. Register a new user
curl -X POST https://api.test.example.com/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@example.com",
    "password": "SecurePass123!",
    "name": "New User"
  }'

# Response:
# {
#   "success": true,
#   "data": {
#     "id": "user_456",
#     "email": "newuser@example.com",
#     "name": "New User"
#   }
# }

# 2. Login with credentials
curl -X POST https://api.test.example.com/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@example.com",
    "password": "SecurePass123!"
  }'

# Response:
# {
#   "success": true,
#   "data": {
#     "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
#     "expiresIn": 3600
#   }
# }

# 3. Access protected resource
curl -X GET https://api.test.example.com/v1/users/me \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### Example 2: Working with Resources

```bash
# Create a new resource
curl -X POST https://api.test.example.com/v1/resources \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Resource",
    "description": "A test resource"
  }'

# List all resources with filtering
curl -X GET "https://api.test.example.com/v1/resources?page=1&perPage=10&search=test" \
  -H "Authorization: Bearer YOUR_TOKEN"

# Update a resource
curl -X PATCH https://api.test.example.com/v1/resources/res_123 \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Updated Resource Name"
  }'

# Delete a resource
curl -X DELETE https://api.test.example.com/v1/resources/res_123 \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Example 3: Error Handling

```bash
# Invalid authentication
curl -X GET https://api.test.example.com/v1/users/me \
  -H "Authorization: Bearer invalid_token"

# Response: 401 Unauthorized
# {
#   "success": false,
#   "error": {
#     "code": "AUTHENTICATION_ERROR",
#     "message": "Invalid or expired token"
#   }
# }

# Validation error
curl -X POST https://api.test.example.com/v1/resources \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": ""
  }'

# Response: 422 Unprocessable Entity
# {
#   "success": false,
#   "error": {
#     "code": "VALIDATION_ERROR",
#     "message": "Invalid input data",
#     "details": [
#       {
#         "field": "name",
#         "message": "Name is required"
#       }
#     ]
#   }
# }
```

## 📚 SDKs & Libraries

> ⚠️ **Missing Information**: SDKs to be developed.

### Official SDKs (Planned)

- **JavaScript/TypeScript**: `npm install @test/api-client`
- **Python**: `pip install test-api-client`
- **Java**: Maven/Gradle dependency
- **Go**: `go get github.com/test/api-client-go`

### JavaScript SDK Example

```javascript
import { TestAPIClient } from '@test/api-client';

const client = new TestAPIClient({
  baseURL: 'https://api.test.example.com/v1',
  apiKey: 'your_api_key'
});

// Authenticate
const { token } = await client.auth.login({
  email: 'user@example.com',
  password: 'password'
});

client.setToken(token);

// Get resources
const resources = await client.resources.list({
  page: 1,
  perPage: 20
});

// Create resource
const newResource = await client.resources.create({
  name: 'New Resource',
  description: 'Description'
});
```

### Python SDK Example

```python
from test_api import TestAPIClient

client = TestAPIClient(
    base_url='https://api.test.example.com/v1',
    api_key='your_api_key'
)

# Authenticate
response = client.auth.login(
    email='user@example.com',
    password='password'
)
client.set_token(response['token'])

# Get resources
resources = client.resources.list(page=1, per_page=20)

# Create resource
new_resource = client.resources.create(
    name='New Resource',
    description='Description'
)
```

## 🔄 Webhooks

> ⚠️ **Missing Information**: Webhook functionality to be implemented.

### Webhook Events (Planned)

- `resource.created`
- `resource.updated`
- `resource.deleted`
- `user.registered`
- `user.deleted`

### Webhook Payload Format

```json
{
  "event": "resource.created",
  "timestamp": "2024-01-01T00:00:00Z",
  "data": {
    "id": "res_123",
    "name": "New Resource"
  }
}
```

## 🧪 Testing the API

### Using Postman

1. Import the Postman collection (to be provided)
2. Set environment variables (base URL, tokens)
3. Run requests

### Using curl

See examples above for curl commands.

### Using HTTPie

```bash
# Install HTTPie
pip install httpie

# Make requests
http POST https://api.test.example.com/v1/auth/login \
  email=user@example.com \
  password=securepassword123

http GET https://api.test.example.com/v1/resources \
  "Authorization:Bearer YOUR_TOKEN"
```

## 📊 API Status & Changelog

### API Status

Check API status at: `https://status.test.example.com` (to be configured)

### Changelog

- **v1.0.0** (TBD): Initial API release

---

## 📞 Support

For API support:
- 📖 Documentation: [https://docs.test.example.com](https://docs.test.example.com)
- 🐛 Report Issues: [GitHub Issues](https://github.com/messier-kevin-cgi/test/issues)
- 💬 Community: [GitHub Discussions](https://github.com/messier-kevin-cgi/test/discussions)

---

**API Version**: 1.0  
**Last Updated**: 2024  
**Status**: 🟡 Template - To be updated with actual implementation