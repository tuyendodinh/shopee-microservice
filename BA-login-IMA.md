# Business Analysis Document

## Project Information

| Item | Value |
|--------|--------|
| Project | Shopee Microservice System |
| Module | Identity & Access Management (IAM) |
| Feature | User Login |
| Version | 1.0 |
| Status | Draft |

---

# 1. Business Context

Shopee là một nền tảng thương mại điện tử cho phép người dùng mua bán sản phẩm trực tuyến. Để sử dụng các chức năng cá nhân hóa và các nghiệp vụ liên quan đến giao dịch, người dùng cần được xác thực danh tính trước khi truy cập hệ thống.

Chức năng Login là điểm khởi đầu của quá trình xác thực, cho phép hệ thống nhận diện người dùng và xác định các quyền truy cập tương ứng với vai trò của họ.

Các chức năng yêu cầu người dùng đăng nhập bao gồm:

- Quản lý thông tin cá nhân
- Quản lý địa chỉ giao hàng
- Quản lý đơn hàng
- Thanh toán
- Theo dõi vận chuyển
- Quản lý gian hàng đối với người bán
- Truy cập các chức năng quản trị đối với quản trị viên

---

# 2. Business Objective

Mục tiêu của chức năng Login:

- Xác thực người dùng hợp lệ.
- Ngăn chặn truy cập trái phép.
- Bảo vệ thông tin cá nhân và dữ liệu giao dịch.
- Cho phép người dùng truy cập các chức năng theo đúng vai trò được cấp.
- Cung cấp trải nghiệm đăng nhập nhanh chóng và thuận tiện.

---

# 3. Actors

## Primary Actors

### Customer

Người sử dụng nền tảng để mua hàng.

### Seller

Người sử dụng nền tảng để bán hàng.

### Administrator

Người quản trị hệ thống.

---

# 4. Scope

## In Scope

- Đăng nhập bằng Email và Password.
- Kiểm tra tính hợp lệ của thông tin đăng nhập.
- Kiểm tra trạng thái tài khoản.
- Ghi nhận lịch sử đăng nhập.
- Cho phép truy cập hệ thống khi xác thực thành công.

## Out of Scope

- Đăng nhập bằng Google.
- Đăng nhập bằng Facebook.
- Đăng nhập bằng Apple ID.
- Đăng nhập bằng số điện thoại.
- Xác thực hai lớp (2FA).
- Đăng nhập bằng sinh trắc học.

Các tính năng trên có thể được triển khai trong các giai đoạn phát triển tiếp theo.

---

# 5. Business Rules

## BR-01

Email sử dụng để đăng nhập phải tồn tại trong hệ thống.

## BR-02

Mật khẩu nhập vào phải khớp với mật khẩu đã được đăng ký.

## BR-03

Người dùng chỉ được phép đăng nhập khi tài khoản ở trạng thái Active.

## BR-04

Người dùng không được phép đăng nhập khi tài khoản ở trạng thái Locked hoặc Suspended.

## BR-05

Sau 5 lần đăng nhập thất bại liên tiếp trong vòng 15 phút, hệ thống sẽ khóa tài khoản tạm thời trong 15 phút.

## BR-06

Mọi lần đăng nhập thành công hoặc thất bại phải được ghi nhận vào nhật ký hệ thống phục vụ mục đích kiểm toán và bảo mật.

## BR-07

Thông báo lỗi không được tiết lộ nguyên nhân cụ thể nhằm hạn chế rủi ro dò quét tài khoản.

Ví dụ:

```text
Email hoặc mật khẩu không chính xác.
```

---

# 6. Use Case

## UC-IAM-001 - User Login

### Use Case Name

User Login

### Goal

Người dùng đăng nhập thành công vào hệ thống.

### Primary Actor

- Customer
- Seller
- Administrator

### Preconditions

- Người dùng đã đăng ký tài khoản.
- Tài khoản tồn tại trong hệ thống.
- Hệ thống đang hoạt động bình thường.

### Trigger

Người dùng nhấn nút **Đăng nhập** trên giao diện.

### Main Flow

#### Step 1

Người dùng nhập:

- Email
- Password

#### Step 2

Người dùng gửi yêu cầu đăng nhập.

#### Step 3

Hệ thống kiểm tra tính hợp lệ của dữ liệu đầu vào.

#### Step 4

Hệ thống xác minh tài khoản tồn tại.

#### Step 5

Hệ thống kiểm tra trạng thái tài khoản.

#### Step 6

Hệ thống xác thực mật khẩu.

#### Step 7

Hệ thống ghi nhận lịch sử đăng nhập thành công.

#### Step 8

Người dùng được chuyển đến trang chủ hoặc trang tương ứng với vai trò của mình.

### Alternative Flow A1

#### Condition

Email không tồn tại.

#### System Response

Hiển thị:

```text
Email hoặc mật khẩu không chính xác.
```

### Alternative Flow A2

#### Condition

Sai mật khẩu.

#### System Response

- Ghi nhận lần đăng nhập thất bại.
- Tăng bộ đếm số lần đăng nhập sai.

Hiển thị:

```text
Email hoặc mật khẩu không chính xác.
```

### Alternative Flow A3

#### Condition

Tài khoản bị khóa.

#### System Response

Hiển thị:

```text
Tài khoản hiện đang bị khóa.
```

### Alternative Flow A4

#### Condition

Tài khoản bị tạm khóa do vượt quá số lần đăng nhập thất bại.

#### System Response

Hiển thị:

```text
Tài khoản tạm thời bị khóa. Vui lòng thử lại sau.
```

### Postconditions

#### Success

- Người dùng được xác thực thành công.
- Người dùng được cấp quyền truy cập hệ thống.

#### Failure

- Người dùng không được phép truy cập hệ thống.

---


# 7. Acceptance Criteria

| ID | Acceptance Criteria |
|------|------|
| AC-01 | Người dùng đăng nhập thành công với thông tin hợp lệ |
| AC-02 | Hệ thống từ chối đăng nhập khi email không tồn tại |
| AC-03 | Hệ thống từ chối đăng nhập khi mật khẩu không chính xác |
| AC-04 | Hệ thống từ chối đăng nhập khi tài khoản bị khóa |
| AC-05 | Hệ thống ghi nhận lịch sử đăng nhập thành công |
| AC-06 | Hệ thống ghi nhận lịch sử đăng nhập thất bại |
| AC-07 | Hệ thống khóa tạm thời tài khoản sau 5 lần đăng nhập thất bại liên tiếp |
| AC-08 | Thời gian phản hồi trung bình nhỏ hơn 2 giây |

---

# 8. Assumptions

- Người dùng đã hoàn thành quá trình đăng ký tài khoản.
- Người dùng sở hữu địa chỉ email hợp lệ.
- Hệ thống email hoạt động bình thường.
- Cơ sở dữ liệu người dùng luôn sẵn sàng phục vụ yêu cầu xác thực.

---
