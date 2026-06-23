## Công nghệ sử dụng trong hệ thống

Hệ thống Shopee được thiết kế theo kiến trúc **Microservice**, trong đó toàn bộ hệ thống được chia thành nhiều service độc lập như User Service, Product Service, Order Service, Payment Service và Shipping Service. Mỗi service đảm nhiệm một nhóm chức năng riêng và có thể được triển khai, mở rộng hoặc bảo trì độc lập với các service khác.

### Tóm tắt vai trò các công nghệ

| Công nghệ | Vai trò |
|-----------|----------|
| Microservice | Chia hệ thống thành các service độc lập |
| Minimal API | Xây dựng API cho từng service, giảm lượng mã nguồn, đơn giản hóa xây dựng các endpoint |
| PostgreSQL | Lưu trữ dữ liệu của từng service |
| gRPC | Giao tiếp đồng bộ giữa các service |
| EDA (Kafka/RabbitMQ) | Giao tiếp bất đồng bộ thông qua event |
| Replica | Bản sao của database chính |
| Sharding | chia nhỏ dữ liệu thành các database khác |
| IAM (JWT RS256) | Xác thực và phân quyền người dùng |
| MinIO/AWS S3 | Lưu trữ các tệp dung lượng lớn như hình ảnh sản phẩm, ảnh đại diện người dùng và tài liệu đính kèm |
| Cache | Cache nơi lưu trữ tạm dữ liệu đã được truy cập trước đó |
| CDN | mạng lưới máy chủ phân tán toàn cầu |
| Cloudflare | dịch vụ bao gồm: CDN, Cache, WAF, DDoS Protection, DNS | 
| HttpOnly | Chỉ HTTP Request mới được sử dụng cookie, cookie lưu token|

# Kiến trúc tổng quan hệ thống Shopee

Hệ thống Shopee được xây dựng theo kiến trúc **Microservice**, trong đó mỗi nghiệp vụ được tách thành một service độc lập nhằm tăng khả năng mở rộng, dễ bảo trì và triển khai riêng biệt. Mỗi service được phát triển bằng **ASP.NET Core Minimal API**, sở hữu cơ sở dữ liệu **PostgreSQL** riêng và giao tiếp với các service khác thông qua **gRPC** hoặc **EDA (Event-Driven Architecture)** tùy theo yêu cầu nghiệp vụ.

## Kiến trúc tổng thể

```text
                                        +------------------+
                                        |     User App     |
                                        +---------+--------+
                                                  |
                                                  v

                                        +------------------+
                                        |    Cloudflare    |
                                        | CDN + WAF + DNS |
                                        +---------+--------+
                                                  |
                                                  v

                                        +------------------+
                                        |   API Gateway    |
                                        +---------+--------+
                                                  |
        -----------------------------------------------------------------------------------
        |                  |                  |                  |               |         |
        v                  v                  v                  v               v         v

+---------------+  +---------------+  +---------------+  +---------------+ +-----------+ +---------------+
|  IAM Service  |  | User Service  |  | Product       |  | Order Service | | Cart      | | Payment       |
| Minimal API   |  | Minimal API   |  | Service       |  | Minimal API   | | Service   | | Service       |
+-------+-------+  +-------+-------+  | Minimal API   |  +-------+-------+ +-----+-----+ +-------+-------+
        |                  |          +-------+-------+          |               |               |
        |                  |                  |                  |               |               |
        -------------------------------------------------------------------------------------------
                                                  |
                                                  |
                                              gRPC Calls
                                                  |
                                                  v

                                 Product <-----> Order <-----> User
                                        \            |
                                         \           |
                                          \          |
                                           \         |
                                            v        v

                                           Payment Service

                                                  |
                                                  |
                                            Publish Event
                                                  |
                                                  v

                                            +-----------+
                                            |   Kafka   |
                                            +-----+-----+
                                                  |
        -----------------------------------------------------------------------------------
        |                    |                     |                     |                |
        v                    v                     v                     v                v

    +---------------+ +----------------+ +----------------+ +----------------+ +----------------+
    | Notification  | | Shipping       | | Analytics      | | Loyalty Point  | | Recommendation |
    | Service       | | Service        | | Service        | | Service        | | Service        |
    +---------------+ +----------------+ +----------------+ +----------------+ +----------------+
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

### Loyalty Point Service

Quản lý điểm thưởng, điểm tích lũy, xu shopee, voucher thưởng.

### Recommendation Service

gợi ý sản phẩm tương tự, liên quan.

---

## Mô tả các công nghệ sử dụng

### Giao tiếp giữa các service

#### Giao tiếp đồng bộ (gRPC)

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

#### Giao tiếp bất đồng bộ (EDA)

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

### Database
#### Replica
Mỗi service sử dụng PostgreSQL với mô hình Primary-Replica.

Primary Database chịu trách nhiệm xử lý các thao tác ghi dữ liệu (INSERT, UPDATE, DELETE), trong khi Replica Database phục vụ các truy vấn đọc dữ liệu (SELECT).

Dữ liệu từ Primary được đồng bộ sang Replica thông qua cơ chế Replication của PostgreSQL.

Mô hình này giúp giảm tải cho cơ sở dữ liệu chính, tăng khả năng mở rộng và cải thiện hiệu năng đọc dữ liệu của hệ thống thương mại điện tử quy mô lớn như Shopee.

```text
                   Product Service

                          |
        ------------------------------------
        |                                  |
        v                                  v

 PostgreSQL Primary              PostgreSQL Replica
      Write                            Read
```

Nguyên tắc quan trọng của Microservice là một service không được truy cập trực tiếp cơ sở dữ liệu của service khác. Thay vào đó, việc trao đổi dữ liệu phải được thực hiện thông qua gRPC hoặc Event.

#### sharding
chia nhỏ dữ liệu thành nhiều database khác nhau

Ví dụ: ban đầu bảng Orders có 10 tỷ record, sau khi sharding
```text
                Order Service

                       |
      ----------------------------------
      |                |               |
      v                v               v

    Shard 1         Shard 2         Shard 3
      |                |               |
   -------          -------         -------
   |     |          |     |         |     |
Primary Replica  Primary Replica  Primary Replica
```

#### MinIO và AWS S3
Hệ thống sử dụng MinIO (tương thích AWS S3) để lưu trữ các tệp dung lượng lớn như hình ảnh sản phẩm, ảnh đại diện người dùng và tài liệu đính kèm.

Thay vì lưu trực tiếp file trong PostgreSQL, hệ thống chỉ lưu đường dẫn (URL) của file trong cơ sở dữ liệu. Điều này giúp giảm kích thước database, tăng hiệu năng truy vấn và tối ưu khả năng mở rộng.

Trong môi trường phát triển, MinIO được sử dụng như một giải pháp thay thế AWS S3 nhờ khả năng triển khai nội bộ và tương thích hoàn toàn với API của S3.
```text
Seller
   |
   v
Product Service
   |
   v
MinIO / S3
   |
   v
Image URL
   |
   v
PostgreSQL
```
#### Cache
Cache nơi lưu trữ tạm dữ liệu đã được truy cập trước đó

### Authentication & Session Security

#### IAM - Identity and accept management
=> phân quyền người dùng
#### SHA-256
=> mã hóa mật khẩu
#### RS256 
Là phần đi cùng JWT 

Sau khi login IAM Service cấp JWT token => Product Service kiểm tra token có giả hay không => việc của RS256

```text
                                                    RS256
                                                      |
                                                      v
                                                -----------------                 
                                                |               |                                                                                          
                                                v               v
                                          private key        public key
                                          IAM service        mọi service
                                                |               |
                                                v               v
                                            Ký token         Xác minh token


User -> IAM service -> Tạo JWT và Ký (private key) -> frontend -> service khác -> kt(public key) -> accept 
```
#### HttpOnly cookie
Nếu không có HttpOnly cookie, JWT token sau khi đăng nhập -> lưu LocalStorage -> JS đọc được -> dễ bị đánh cắp

HttpOnly cookie -> Browser tự lưu token -> JS không đọc được, không sửa được -> giúp chống token theft

Cookie được cấu hình với các thuộc tính:

- HttpOnly: Ngăn JavaScript truy cập token.
- Secure: Chỉ gửi cookie qua HTTPS.
- SameSite=Strict: Hạn chế tấn công CSRF.


### CDN - Content delivery network (Mạng lưới máy chủ phân tán toàn cầu)
Ảnh nằm ở Singapore -> người Việt tải ảnh 
```text
Singapore
     |
     v
CDN Node VN

Singapore
     |
     v
CDN Node Thailand

Singapore
     |
     v
CDN Node Indonesia
```
### Cloudflare

Là một dịch vụ bao gồm: CDN, Cache, WAF, DDoS Protection, DNS

### Rate Limit & Threshold & Quota

Rate limit: giới hạn số lương request trong một khoảng thời gian

```text
        Rate Limit: 100 request/phút
                    |
                    v
           ------------------
           |                |
           v                v
       80 request       200 request
           |                |
           v                v
       cho phép            chặn

Ngăn: Bot, Spam, DDoS nhẹ, Brute Force
```

Threshold: ngưỡng cảnh báo

```text
        Threshold: 800 request/phút -> vượt 800 -> cảnh báo
```

Quota: tổng lượng tài nguyên được phép dùng





