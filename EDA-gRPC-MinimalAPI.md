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
