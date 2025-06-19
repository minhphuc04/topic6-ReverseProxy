
# Triển khai mô hình Reverse Proxy kết hợp giữa Nginx và Apache từ LEMP

## I. Giới thiệu

Tiếp tục từ mô hình **LEMP** (Linux – Nginx – MySQL – PHP) đã cài đặt trước đó, bài này xây dựng mô hình **Reverse Proxy** kết hợp **Nginx** và **Apache** để chạy 2 website với yêu cầu như sau:

---

## II. Thành phần công nghệ sử dụng

- **Nginx**: Web Server đứng trước, làm reverse proxy
- **Apache2**: Web Server phía sau, xử lý động
- **PHP**: Ngôn ngữ xử lý server-side
- **MySQL**: Hệ quản trị cơ sở dữ liệu
- **phpMyAdmin**: Giao diện quản lý CSDL
- **ZeroSSL**: Cung cấp chứng chỉ SSL cho các domain

---

## III. Yêu cầu triển khai

### 1. Cấu trúc Reverse Proxy

- Mô hình kết hợp giữa:
  - Nginx: reverse proxy xử lý tầng ngoài cùng (HTTP/HTTPS)
  - Apache: xử lý nội dung động (PHP/WordPress/Laravel)
- Reverse theo cả 2 giao thức:
  - `http (nginx) → http (apache)`
  - `https (nginx) → https (apache)`

---

### 2. Website sử dụng

- Sử dụng **Virtual Hosts** cho từng domain
- Gắn vào 2 source code tương ứng:

| Website           | Loại      | Domain                        |
|------------------|-----------|-------------------------------|
| Website WordPress | CMS       | `mphuc.wp.vietnix.tech`       |
| Website Laravel   | Framework | `mphuc.laravel.vietnix.tech`  |

---

### 3. Yêu cầu bảo mật

- Áp dụng chứng chỉ **SSL từ ZeroSSL** cho cả hai domain
- Cả hai website đều hoạt động trên:
  - `HTTP` (`port 80`)
  - `HTTPS` (`port 443`)

---

### 4. Default Vhost

- Thiết lập **default vhost** để xử lý mọi domain lạ hoặc IP trỏ trực tiếp về VPS:

```nginx
server {
    listen 80 default_server;
    listen 443 ssl default_server;
    server_name _;

    ssl_certificate     /etc/ssl/certs/zerossl_default.crt;
    ssl_certificate_key /etc/ssl/private/zerossl_default.key;

    location / {
        return 403 "Access Denied.";
    }
}
```

---


## IV. Giải thích: Vì sao Nginx đứng trước Apache?

Việc triển khai mô hình Reverse Proxy với Nginx đứng trước Apache là một lựa chọn kiến trúc phổ biến và có lý do rõ ràng về mặt hiệu năng, bảo mật và khả năng mở rộng.

### 1. Hiệu năng cao hơn

Nginx được thiết kế theo mô hình **event-driven, non-blocking I/O**, giúp xử lý hàng ngàn kết nối đồng thời với mức tài nguyên thấp hơn so với Apache.

Khi đứng trước Apache, Nginx xử lý cực nhanh các request tĩnh như:

- Hình ảnh (JPG, PNG, SVG),
- CSS,
- JavaScript,
- Video...

Chỉ các request động (PHP) mới được chuyển ngược về Apache xử lý.

### 2. Tăng cường bảo mật

Nginx đóng vai trò như một lớp bảo vệ đầu tiên (frontend), giúp:

- **Chặn truy cập trái phép**, spam, DDoS cơ bản.
- **Ẩn toàn bộ hệ thống phía sau** như Apache, PHP, CSDL.
- **SSL termination**: xử lý chứng chỉ SSL/HTTPS tại Nginx giúp giảm tải và tránh cấu hình SSL phức tạp trên Apache.

### 3. Khả năng mở rộng và phân tán

Nginx dễ dàng đóng vai trò:

- **Load Balancer** phân phối request tới nhiều backend như Apache, NodeJS...
- **Gateway** chia tầng rõ ràng giữa API backend và frontend.
- Có thể kết hợp cùng Docker hoặc Kubernetes để scale linh hoạt.

### 4. Đơn giản hóa cấu hình đa miền

Nginx hỗ trợ **Virtual Host (server block)** rất linh hoạt:

- Quản lý nhiều domain trên cùng 1 server (WordPress, Laravel...).
- Gắn mỗi domain với SSL riêng biệt từ ZeroSSL, Let’s Encrypt...
- Cấu hình HTTP/2, HSTS, cache tĩnh trực tiếp.

### 5. Tận dụng điểm mạnh của NGINX khi đứng trước Apache

| Thành phần     | Nhiệm vụ chính                                |
|----------------|------------------------------------------------|
| **NGINX**      | Xử lý tĩnh (ảnh, JS, CSS), SSL, phân luồng    |
| **Apache**     | Xử lý động (PHP, Laravel, WordPress)          |

**Thực tế mô hình đã triển khai:**

- Client gửi HTTP/HTTPS → Nginx (port 80/443).
- Nginx lọc, giải mã SSL, chuyển tiếp:
  - HTTP → Apache:8081
  - HTTPS → Apache:8443
- Apache xử lý nội dung động, trả kết quả lại cho Nginx → trả về client.

>  Như vậy, toàn bộ **ưu điểm về hiệu năng và bảo mật của Nginx** được tận dụng tối đa, trong khi **Apache tập trung xử lý nội dung động** như Laravel và WordPress — vốn là thế mạnh của Apache + PHP.
---
## V. Kết quả đạt được

### 1. Mô hình hoạt động hoàn chỉnh

- Nginx reverse proxy cho toàn bộ truy cập.
- Apache xử lý nội dung động (PHP/WordPress/Laravel).
- Hệ thống phân tầng rõ ràng, tối ưu hiệu suất.

### 2. Hai website hoạt động độc lập

| Domain                            | Trạng thái | Ghi chú                     |
|-----------------------------------|------------|------------------------------|
| https://mphuc.wp.vietnix.tech     | Online     | WordPress qua Nginx + Apache |
| https://mphuc.laravel.vietnix.tech| Online     | Laravel qua Nginx + Apache   |

### 3. Hỗ trợ đầy đủ HTTP và HTTPS

- Tự động redirect từ HTTP sang HTTPS nếu cấu hình.
- SSL từ ZeroSSL được trình duyệt tin cậy.

### 4. Default vhost xử lý đúng

- Các truy cập IP/domain lạ đều bị từ chối.
- Tránh rò rỉ thông tin hệ thống.

### 5. phpMyAdmin truy cập nội bộ

- phpMyAdmin cài đặt thành công.
- Có thể giới hạn truy cập nội bộ qua domain riêng.

---

## VI. Kết luận

Mô hình Reverse Proxy với Nginx đứng trước Apache trong môi trường LEMP mang lại:

- Tối ưu hiệu suất và bảo mật.
- Dễ cấu hình nhiều website và chứng chỉ SSL.
- Dễ mở rộng và quản lý trong môi trường thực tế.

Phù hợp cho hệ thống nhỏ đến trung bình cần hiệu suất cao và bảo mật tốt.

---

## VII. So sánh chi tiết: Nginx và Apache

| Tiêu chí                | **Nginx**                           | **Apache**                           |
|-------------------------|-------------------------------------|--------------------------------------|
| Kiến trúc chính         | Event-driven, non-blocking I/O     | Process/thread-based (blocking I/O) |
| Tốc độ xử lý tĩnh       | Rất nhanh                          | Chậm hơn                             |
| Tài nguyên              | Ít, nhẹ                            | Tốn RAM/CPU nhiều hơn                |
| Thích hợp cho           | Proxy, load balancing, static file | Xử lý PHP, module dynamic            |

---

### Ưu điểm của Nginx

- Xử lý nội dung tĩnh (ảnh, JS, CSS) cực kỳ nhanh.
- Làm reverse proxy và load balancer hiệu quả.
- Có thể termination SSL ngay tại tầng ngoài.
- Dùng ít RAM, CPU.

###  Nhược điểm

- Không hỗ trợ `.htaccess` (thiếu linh hoạt với cấu hình thư mục).
- Cấu hình phức tạp hơn với người mới bắt đầu.
- Thiếu khả năng xử lý dynamic content trực tiếp (cần kết hợp với PHP-FPM hoặc backend server như Apache).
- Debug lỗi phức tạp hơn nếu không quen cấu trúc log.


### Ưu điểm của Apache

- Tích hợp tốt với các module PHP (mod_php).
- Hỗ trợ `.htaccess` cấu hình từng thư mục.
- Phù hợp với các CMS như WordPress, Laravel.

### Nhược điểm

- Kiến trúc **process/thread-based**, nên tiêu tốn nhiều tài nguyên hơn (RAM, CPU).
- **Xử lý nội dung tĩnh kém hơn Nginx**.
- Hiệu suất giảm đáng kể khi có nhiều kết nối đồng thời.
- Không tối ưu khi dùng như load balancer.

---

## VIII. Tại sao kết hợp cả hai?

| Kết hợp                 | Lợi ích                                                  |
|-------------------------|----------------------------------------------------------|
| `Nginx (proxy)`         | Xử lý SSL, tĩnh, redirect, load balancing                |
| `Apache (backend)`      | Xử lý nội dung động, PHP, CMS Laravel/WordPress         |
| Nginx → Apache          | Giao tiếp ngược proxy_pass để tận dụng sức mạnh riêng   |

=> **Tận dụng điểm mạnh của cả hai**, xây dựng hệ thống bảo mật – hiệu năng cao – dễ mở rộng.
