# API Contract

## Module

Identity & Access Management (IAM)

## Feature

User Login

## Version

1.0

---

# 1. Overview

API Login cho phép người dùng xác thực danh tính bằng Email và Password.

Sau khi xác thực thành công, hệ thống cấp quyền truy cập thông qua Access Token và Refresh Token.

---

# 2. Endpoint Information

| Field | Value |
|---------|---------|
| Method | POST |
| Endpoint | `/api/v1/auth/login` |
| Authentication | Not Required |
| Content-Type | application/json |

---

# 3. Request

## Request Body

```json
{
  "email": "user@example.com",
  "password": "Password@123"
}
```
