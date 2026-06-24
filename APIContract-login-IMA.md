# API Contract

## Module

Identity & Access Management (IAM)

## Domain

Authentication & Authorization

## Version

1.0

---

# 1. Overview

Tài liệu này mô tả các API liên quan đến xác thực người dùng trong hệ thống Shopee Microservice.

Các chức năng bao gồm:

- Login
- Refresh Access Token
- Logout
- Get Current User Profile

---

# 2. API Standards

## Base URL

```http
https://api.shopee.local
```

## API Version

```http
/api/v1
```

## Content Type

```http
application/json
```

## Time Format

```text
ISO 8601 (UTC)
```

Ví dụ:

```text
2026-06-23T10:00:00Z
```

---

# 3. Authentication APIs

---

# 3.1 Login

Cho phép người dùng đăng nhập bằng Email và Password.

## Endpoint

```http
POST /api/v1/auth/login
```

## Authentication

Not Required

## Request Body

```json
{
  "email": "user@example.com",
  "password": "Password@123"
}
```

### Request Fields

| Field | Type | Required | Description |
|---------|---------|---------|---------|
| email | string | Yes | User email |
| password | string | Yes | User password |

---

### Validation Rules

#### email

- Required
- Valid email format
- Maximum 255 characters

#### password

- Required
- Minimum 8 characters
- Maximum 100 characters

---

## Success Response

### HTTP Status

```http
200 OK
```

### Response Body

```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIs...",
  "refreshToken": "c7f88d9d-cc29-4d8a-8d57-a2f1f8c18e67",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

### Response Fields

| Field | Type | Description |
|---------|---------|---------|
| accessToken | string | JWT Access Token |
| refreshToken | string | Refresh Token |
| tokenType | string | Authentication scheme |
| expiresIn | integer | Expiration time (seconds) |

---

## Business Rules

| Rule ID | Description |
|----------|-------------|
| BR-01 | Email must exist |
| BR-02 | Password must match |
| BR-03 | User status must be Active |
| BR-04 | Locked account cannot login |
| BR-05 | Suspended account cannot login |

---

# 3.2 Refresh Token

Cho phép lấy Access Token mới khi Access Token hết hạn.

## Endpoint

```http
POST /api/v1/auth/refresh
```

## Authentication

Not Required

## Request Body

```json
{
  "refreshToken": "c7f88d9d-cc29-4d8a-8d57-a2f1f8c18e67"
}
```

---

## Success Response

### HTTP Status

```http
200 OK
```

### Response Body

```json
{
  "accessToken": "new-access-token",
  "refreshToken": "new-refresh-token",
  "expiresIn": 3600
}
```

---

## Business Rules

| Rule ID | Description |
|----------|-------------|
| BR-06 | Refresh token must exist |
| BR-07 | Refresh token must not be revoked |
| BR-08 | Refresh token must not be expired |
| BR-09 | New refresh token must be generated (token rotation) |

---

# 3.3 Logout

Cho phép người dùng đăng xuất khỏi hệ thống.

## Endpoint

```http
POST /api/v1/auth/logout
```

## Authentication

Required

## Headers

```http
Authorization: Bearer {accessToken}
```

---

## Request Body

```json
{
  "refreshToken": "c7f88d9d-cc29-4d8a-8d57-a2f1f8c18e67"
}
```

---

## Success Response

### HTTP Status

```http
200 OK
```

### Response Body

```json
{
  "message": "Logout successfully"
}
```

---

## Business Rules

| Rule ID | Description |
|----------|-------------|
| BR-10 | Refresh token must be revoked |
| BR-11 | User session must be terminated |

---

# 3.4 Get Current User

Lấy thông tin người dùng đang đăng nhập.

## Endpoint

```http
GET /api/v1/auth/me
```

## Authentication

Required

## Headers

```http
Authorization: Bearer {accessToken}
```

---

## Success Response

### HTTP Status

```http
200 OK
```

### Response Body

```json
{
  "id": "0f3dba63-8f6c-4a95-b12f-2e9cb1f71f2d",
  "email": "user@example.com",
  "fullName": "Nguyen Van A",
  "roles": [
    "Customer"
  ]
}
```

---

# 4. Error Handling

## Standard Error Response

```json
{
  "code": "ERROR_CODE",
  "message": "Human readable message"
}
```

---

## Error Codes

| Code | HTTP Status | Description |
|---------|---------|---------|
| VALIDATION_ERROR | 400 | Invalid request |
| INVALID_CREDENTIALS | 401 | Wrong email or password |
| UNAUTHORIZED | 401 | Missing token |
| FORBIDDEN | 403 | Permission denied |
| ACCOUNT_LOCKED | 403 | User account locked |
| ACCOUNT_SUSPENDED | 403 | User account suspended |
| TOKEN_EXPIRED | 401 | Access token expired |
| INVALID_REFRESH_TOKEN | 401 | Invalid refresh token |
| RATE_LIMIT_EXCEEDED | 429 | Too many requests |
| INTERNAL_SERVER_ERROR | 500 | Unexpected error |

---

# 5. Rate Limiting

## Login

| Property | Value |
|------------|------------|
| Window | 1 minute |
| Limit | 10 requests |
| Scope | Per IP |

---

## Refresh Token

| Property | Value |
|------------|------------|
| Window | 1 minute |
| Limit | 30 requests |
| Scope | Per User |

---

# 6. Audit Logging

Hệ thống phải ghi nhận các thông tin sau:

- User Id
- Email
- Login Time
- Client IP
- User Agent
- Login Result
- Failure Reason

---

# 7. Security Considerations

## Access Token

- JWT
- Signed using RS256
- Expiration: 60 minutes

## Refresh Token

- Random UUID
- Stored in database
- Revocable
- Expiration: 30 days

## Transport Security

- HTTPS only

## Cookie Security

Nếu sử dụng Cookie:

- HttpOnly
- Secure
- SameSite=Strict

---

# 8. Sequence Flow

```text
Client
   |
   | POST /auth/login
   |
   v

IAM Service
   |
   | Validate Request
   |
   | Verify Credential
   |
   | Generate Token
   |
   v

Client

```
