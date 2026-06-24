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

# 4. Roles

Lưu danh sách vai trò.

## Table

roles

### Columns

| Column | Type | Nullable | Description |
|----------|----------|----------|----------|
| id | UUID | No | Primary Key |
| name | VARCHAR(100) | No | Role name |
| description | TEXT | Yes | Description |

---

### Sample Data

```text
ADMIN
CUSTOMER
SELLER
```

---

# 5. Permissions

Lưu danh sách quyền.

## Table

permissions

### Columns

| Column | Type | Nullable | Description |
|----------|----------|----------|----------|
| id | UUID | No | Primary Key |
| name | VARCHAR(100) | No | Permission name |
| description | TEXT | Yes | Description |

---

### Sample Data

```text
USER_READ
USER_WRITE

PRODUCT_READ
PRODUCT_WRITE

ORDER_READ
ORDER_WRITE
```

---

# 6. User Roles

Quan hệ N-N giữa User và Role.

## Table

user_roles

### Columns

| Column | Type |
|----------|----------|
| user_id | UUID |
| role_id | UUID |

---

### Constraints

```sql
PRIMARY KEY (user_id, role_id)

FOREIGN KEY (user_id)
REFERENCES users(id)

FOREIGN KEY (role_id)
REFERENCES roles(id)
```

---
