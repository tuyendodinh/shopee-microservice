# Microservice Architecture in Shopee

## 1. Giới thiệu

Shopee là một trong những nền tảng thương mại điện tử lớn nhất Đông Nam Á, phục vụ hàng triệu người dùng và xử lý khối lượng giao dịch rất lớn mỗi ngày.

Để đáp ứng yêu cầu về khả năng mở rộng, tính sẵn sàng và dễ bảo trì, Shopee áp dụng kiến trúc Microservice thay vì kiến trúc Monolithic truyền thống.

Microservice là kiến trúc chia hệ thống lớn thành nhiều dịch vụ nhỏ, độc lập. Mỗi dịch vụ đảm nhiệm một chức năng riêng và giao tiếp với nhau thông qua API.

## 2. Kiến trúc tổng quan

```text
                         +----------------+
                         |     User       |
                         +--------+-------+
                                  |
                                  v
                        +------------------+
                        |   API Gateway    |
                        +--------+---------+
                                 |
        -----------------------------------------------------
        |           |            |           |             |
        v           v            v           v             v

 +-------------+ +-------------+ +-------------+ +-------------+ +-------------+
 | User        | | Product     | | Order       | | Payment     | | Shipping    |
 | Service     | | Service     | | Service     | | Service     | | Service     |
 +------+------+ +------+------+ +------+------+ +------+------+ +------+------+
        |               |               |               |               |
        v               v               v               v               v

 +-------------+ +-------------+ +-------------+ +-------------+ +-------------+
 | User DB     | | Product DB  | | Order DB    | | Payment DB  | | Shipping DB |
 +-------------+ +-------------+ +-------------+ +-------------+ +-------------+
```

## 3. Các thành phần chính

### 3.1 User Service

Chịu trách nhiệm quản lý người dùng.

Chức năng:

- Đăng ký tài khoản
- Đăng nhập
- Quản lý hồ sơ cá nhân
- Xác thực người dùng

Dữ liệu lưu trữ:

- User ID
- Username
- Password
- Địa chỉ giao hàng

### 3.2 Product Service

Quản lý toàn bộ thông tin sản phẩm.

Chức năng:

- Thêm sản phẩm
- Cập nhật sản phẩm
- Quản lý tồn kho
- Tìm kiếm sản phẩm

Dữ liệu lưu trữ:

- Product ID
- Tên sản phẩm
- Giá bán
- Số lượng tồn kho

### 3.3 Order Service

Quản lý đơn hàng.

Chức năng:

- Tạo đơn hàng
- Hủy đơn hàng
- Cập nhật trạng thái đơn hàng

Dữ liệu lưu trữ:

- Order ID
- User ID
- Product ID
- Trạng thái đơn hàng

### 3.4 Payment Service

Xử lý thanh toán.

Chức năng:

- Thanh toán bằng ví điện tử
- Thanh toán bằng thẻ ngân hàng
- Hoàn tiền

Dữ liệu lưu trữ:

- Payment ID
- Order ID
- Số tiền thanh toán
- Trạng thái giao dịch

### 3.5 Shipping Service

Quản lý vận chuyển.

Chức năng:

- Tạo vận đơn
- Theo dõi trạng thái giao hàng
- Cập nhật vị trí đơn hàng

Dữ liệu lưu trữ:

- Shipping ID
- Order ID
- Đơn vị vận chuyển
- Trạng thái giao hàng

## 4. Luồng xử lý đặt hàng

Khi khách hàng mua một sản phẩm trên Shopee:

### Bước 1

Người dùng nhấn nút "Mua ngay".

### Bước 2

Yêu cầu được gửi tới Order Service.

### Bước 3

Order Service gọi Product Service để kiểm tra tồn kho.

```text
Order Service
      |
      v
Product Service
```

### Bước 4

Nếu còn hàng, Order Service tạo đơn hàng.

### Bước 5

Order Service gọi Payment Service để xử lý thanh toán.

```text
Order Service
      |
      v
Payment Service
```

### Bước 6

Sau khi thanh toán thành công, Shipping Service tạo vận đơn.

```text
Payment Service
      |
      v
Shipping Service
```

### Bước 7

Thông tin đơn hàng được trả về cho người dùng.

```text
Khách hàng
      |
      v
Đặt hàng
      |
      v
Order Service
      |
      +------> Product Service
      |
      +------> Payment Service
      |
      +------> Shipping Service
      |
      v
Đơn hàng thành công
```

## 5. Ưu điểm của Microservice

### Khả năng mở rộng

Mỗi service có thể mở rộng độc lập.

Ví dụ:

Nếu lượng thanh toán tăng mạnh trong ngày 11/11, Shopee chỉ cần tăng tài nguyên cho Payment Service.

### Dễ bảo trì

Lỗi ở một service không ảnh hưởng trực tiếp đến toàn bộ hệ thống.

### Triển khai độc lập

Có thể cập nhật Payment Service mà không cần dừng User Service hoặc Product Service.

## 6. Nhược điểm của Microservice

### Hệ thống phức tạp

Cần quản lý nhiều service khác nhau.

### Khó giám sát

Khó theo dõi lỗi khi một giao dịch đi qua nhiều service.

### Tăng chi phí vận hành

Cần nhiều máy chủ và công cụ quản lý hơn so với Monolithic.

## 7. Kết luận

Kiến trúc Microservice giúp Shopee đáp ứng lượng truy cập và giao dịch rất lớn bằng cách chia hệ thống thành các dịch vụ nhỏ độc lập như User Service, Product Service, Order Service, Payment Service và Shipping Service.

Nhờ đó hệ thống có khả năng mở rộng tốt, dễ bảo trì và phù hợp với các nền tảng thương mại điện tử quy mô lớn.
