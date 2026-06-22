## Công nghệ sử dụng trong hệ thống

Hệ thống Shopee được thiết kế theo kiến trúc **Microservice**, trong đó toàn bộ hệ thống được chia thành nhiều service độc lập như User Service, Product Service, Order Service, Payment Service và Shipping Service. Mỗi service đảm nhiệm một nhóm chức năng riêng và có thể được triển khai, mở rộng hoặc bảo trì độc lập với các service khác.

Để xây dựng các API cho từng service, hệ thống sử dụng **ASP.NET Core Minimal API**. Minimal API giúp giảm lượng mã nguồn cần viết, đơn giản hóa việc xây dựng endpoint và phù hợp với các microservice có phạm vi chức năng nhỏ.

Mỗi service sở hữu một cơ sở dữ liệu riêng được xây dựng trên **PostgreSQL**. Việc tách biệt cơ sở dữ liệu giúp các service độc lập với nhau, hạn chế phụ thuộc trực tiếp vào dữ liệu của service khác và tăng khả năng mở rộng của hệ thống.

Đối với các nghiệp vụ cần phản hồi ngay lập tức, chẳng hạn như kiểm tra thông tin người dùng, xác thực đơn hàng hoặc kiểm tra tồn kho sản phẩm, các service giao tiếp với nhau thông qua **gRPC**. Đây là cơ chế giao tiếp đồng bộ (synchronous communication) có hiệu năng cao, sử dụng HTTP/2 và Protocol Buffers để giảm kích thước dữ liệu truyền tải và tăng tốc độ xử lý.

Ngoài ra, hệ thống còn áp dụng **Event-Driven Architecture (EDA)** kết hợp với Kafka hoặc RabbitMQ để xử lý các nghiệp vụ bất đồng bộ. Khi một sự kiện xảy ra, ví dụ đơn hàng được tạo thành công, Order Service sẽ phát sinh sự kiện `OrderCreated`. Các service khác như Payment Service, Notification Service hoặc Analytics Service có thể lắng nghe và xử lý sự kiện này mà không cần giao tiếp trực tiếp với Order Service. Điều này giúp giảm sự phụ thuộc giữa các service, tăng khả năng mở rộng và nâng cao độ ổn định của hệ thống.

### Tóm tắt vai trò các công nghệ

| Công nghệ | Vai trò |
|-----------|----------|
| Microservice | Chia hệ thống thành các service độc lập |
| Minimal API | Xây dựng API cho từng service |
| PostgreSQL | Lưu trữ dữ liệu của từng service |
| gRPC | Giao tiếp đồng bộ giữa các service |
| EDA (Kafka/RabbitMQ) | Giao tiếp bất đồng bộ thông qua event |

# Kiến trúc tổng quan hệ thống Shopee

Hệ thống Shopee được xây dựng theo kiến trúc **Microservice**, trong đó mỗi nghiệp vụ được tách thành một service độc lập nhằm tăng khả năng mở rộng, dễ bảo trì và triển khai riêng biệt. Mỗi service được phát triển bằng **ASP.NET Core Minimal API**, sở hữu cơ sở dữ liệu **PostgreSQL** riêng và giao tiếp với các service khác thông qua **gRPC** hoặc **EDA (Event-Driven Architecture)** tùy theo yêu cầu nghiệp vụ.

## Kiến trúc tổng thể

```text
                           +----------------+
                           |    Frontend    |
                           +--------+-------+
                                    |
                                    v
                           +----------------+
                           |  API Gateway   |
                           +--------+-------+
                                    |
        ----------------------------------------------------------------
        |                |                |                |            |
        v                v                v                v            v

+---------------+ +---------------+ +---------------+ +---------------+ +---------------+
| User Service  | | Product       | | Order Service | | Payment       | | Shipping      |
| Minimal API   | | Service       | | Minimal API   | | Service       | | Service       |
| PostgreSQL    | | Minimal API   | | PostgreSQL    | | Minimal API   | | Minimal API   |
+-------+-------+ | PostgreSQL    | +-------+-------+ | PostgreSQL    | | PostgreSQL    |
        |         +-------+-------+         |         +-------+-------+ +-------+-------+
        |                 ^                 |                 ^
        |                 |                 |                 |
        +-----------------+------ gRPC -----+-----------------+
                          |
                          v

                    +------------+
                    |   Kafka    |
                    +------------+
                          |
          -----------------------------------------
          |                    |                  |
          v                    v                  v

+----------------+  +----------------+  +----------------+
| Notification   |  | Analytics      |  | Loyalty Point  |
| Service        |  | Service        |  | Service        |
+----------------+  +----------------+  +----------------+
```

## Thành phần chính

### API Gateway

API Gateway là điểm tiếp nhận tất cả request từ phía người dùng. Gateway chịu trách nhiệm định tuyến request đến đúng service, xác thực người dùng và quản lý lưu lượng truy cập.

### User Service

Quản lý thông tin người dùng, đăng ký, đăng nhập và hồ sơ cá nhân. Service được xây dựng bằng Minimal API và sử dụng PostgreSQL để lưu trữ dữ liệu người dùng.

### Product Service

Quản lý sản phẩm, danh mục, giá bán và tồn kho. Product Service cung cấp các API và gRPC endpoint để các service khác kiểm tra thông tin sản phẩm.

### Order Service

Là trung tâm điều phối quy trình đặt hàng. Service này tiếp nhận yêu cầu mua hàng, kiểm tra người dùng và sản phẩm thông qua gRPC, sau đó tạo đơn hàng và phát sinh các sự kiện (event) cho các service khác xử lý.

### Payment Service

Xử lý các giao dịch thanh toán. Service này lắng nghe các sự kiện liên quan đến đơn hàng và cập nhật trạng thái thanh toán tương ứng.

### Shipping Service

Quản lý vận chuyển và tạo vận đơn sau khi thanh toán thành công.

### Notification Service

Nhận các sự kiện từ Kafka để gửi email, SMS hoặc thông báo trong ứng dụng cho người dùng.

### Analytics Service

Thu thập dữ liệu sự kiện từ hệ thống nhằm phục vụ báo cáo và phân tích kinh doanh.

---

## Giao tiếp giữa các service

### Giao tiếp đồng bộ (gRPC)

Các nghiệp vụ yêu cầu phản hồi ngay lập tức sử dụng gRPC.

Ví dụ:

- Kiểm tra thông tin người dùng.
- Kiểm tra tồn kho sản phẩm.
- Lấy thông tin sản phẩm.
- Xác thực trạng thái đơn hàng.

```text
Order Service
      |
      | gRPC
      v
Product Service
```

### Giao tiếp bất đồng bộ (EDA)

Các nghiệp vụ không yêu cầu phản hồi ngay sử dụng Kafka.

Ví dụ:

```text
Order Service
      |
      v
OrderCreated Event
      |
      v
Kafka
      |
-------------------------
|           |           |
v           v           v

Payment   Analytics   Notification
Service   Service     Service
```

Khi một đơn hàng được tạo thành công, Order Service phát sinh sự kiện `OrderCreated`. Các service khác sẽ tự động lắng nghe và xử lý nghiệp vụ của mình mà không cần được gọi trực tiếp.

---

## Database

Mỗi service sở hữu cơ sở dữ liệu PostgreSQL riêng.

```text
User Service
    |
    v
PostgreSQL

Product Service
    |
    v
PostgreSQL

Order Service
    |
    v
PostgreSQL

Payment Service
    |
    v
PostgreSQL
```

Nguyên tắc quan trọng của Microservice là một service không được truy cập trực tiếp cơ sở dữ liệu của service khác. Thay vào đó, việc trao đổi dữ liệu phải được thực hiện thông qua gRPC hoặc Event.
