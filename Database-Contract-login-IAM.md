# Database Schema

## Module

Identity & Access Management (IAM)

## Database

PostgreSQL

## Version

1.0

---

# 1. Overview

IAM Database chịu trách nhiệm lưu trữ:

- User Accounts
- Roles
- Permissions
- User Role Mapping
- Refresh Tokens
- Login Audit Logs

---

# 2. Entity Relationship Diagram (ERD)

```text
Users
  |
  |
UserRoles
  |
  |
Roles
  |
  |
RolePermissions
  |
  |
Permissions

Users
  |
  |
RefreshTokens

Users
  |
  |
LoginAuditLogs
```

---

# 3. Users

Lưu thông tin tài khoản người dùng.

## Table

users

### Columns

| Column | Type | Nullable | Description |
|----------|----------|----------|----------|
| id | UUID | No | Primary Key |
| email | VARCHAR(255) | No | User email |
| password_hash | TEXT | No | Password hash |
| full_name | VARCHAR(255) | Yes | Full name |
| status | VARCHAR(50) | No | Account status |
| created_at | TIMESTAMP | No | Created time |
| updated_at | TIMESTAMP | No | Updated time |

---

### Constraints

```sql
PRIMARY KEY (id)

UNIQUE(email)
```

---

### Status Values

```text
ACTIVE
LOCKED
SUSPENDED
```

---


