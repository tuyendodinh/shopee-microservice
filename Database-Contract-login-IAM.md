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

# 7. Role Permissions

Quan hệ N-N giữa Role và Permission.

## Table

role_permissions

### Columns

| Column | Type |
|----------|----------|
| role_id | UUID |
| permission_id | UUID |

---

### Constraints

```sql
PRIMARY KEY (role_id, permission_id)
```

---

# 8. Refresh Tokens

Lưu Refresh Token phục vụ đăng nhập dài hạn.

## Table

refresh_tokens

### Columns

| Column | Type | Nullable |
|----------|----------|----------|
| id | UUID | No |
| user_id | UUID | No |
| token | TEXT | No |
| expires_at | TIMESTAMP | No |
| revoked | BOOLEAN | No |
| created_at | TIMESTAMP | No |

---

### Constraints

```sql
PRIMARY KEY(id)

FOREIGN KEY(user_id)
REFERENCES users(id)
```

---

### Business Rules

- Một User có thể có nhiều Refresh Token.
- Refresh Token có thể bị revoke.
- Refresh Token có thời gian hết hạn.

---

# 9. Login Audit Logs

Lưu lịch sử đăng nhập.

## Table

login_audit_logs

### Columns

| Column | Type |
|----------|----------|
| id | UUID |
| user_id | UUID |
| email | VARCHAR(255) |
| ip_address | VARCHAR(100) |
| user_agent | TEXT |
| login_result | VARCHAR(50) |
| failure_reason | VARCHAR(255) |
| created_at | TIMESTAMP |

---

### Login Result

```text
SUCCESS
FAILED
LOCKED
```

---

# 10. Index Strategy

## Users

```sql
CREATE UNIQUE INDEX idx_users_email
ON users(email);
```

---

## Refresh Tokens

```sql
CREATE INDEX idx_refresh_tokens_user
ON refresh_tokens(user_id);
```

---

## Login Audit Logs

```sql
CREATE INDEX idx_login_logs_user
ON login_audit_logs(user_id);

CREATE INDEX idx_login_logs_created_at
ON login_audit_logs(created_at);
```

---

# 11. Data Retention Policy

## Login Audit Logs

Retention:

```text
180 days
```

Sau thời gian này dữ liệu sẽ được archive hoặc xóa.

---

# 12. Future Enhancements

Các bảng có thể bổ sung trong tương lai:

- user_sessions
- otp_codes
- password_reset_tokens
- device_registrations
- mfa_settings




