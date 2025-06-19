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

## IV. Giải thích: Vì sao Nginx đứng trước Apache?

Việc triển khai mô hình Reverse Proxy với Nginx đứng trước Apache là một lựa chọn kiến trúc phổ biến và có lý do rõ ràng về mặt hiệu năng, bảo mật và khả năng mở rộng.

### 1. Hiệu năng cao hơn

Nginx được thiết kế theo mô hình **event-driven, non-blocking I/O**, giúp xử lý hàng ngàn kết nối đồng thời với mức tài nguyên thấp hơn so với Apache.

Khi đứng trước Apache, Nginx xử lý các kết nối tĩnh như CSS, JS, ảnh, video cực kỳ hiệu quả và chỉ proxy các yêu cầu động (PHP) về Apache xử lý.

### 2. Tăng cường bảo mật

Nginx đóng vai trò như một lớp bảo vệ đầu tiên (frontend), giúp:

- Chặn truy cập trái phép, spam, DDoS cơ bản.
- Ẩn thông tin chi tiết hệ thống Apache khỏi client.
- SSL termination tập trung tại Nginx giúp đơn giản hóa cấu hình và bảo vệ.

### 3. Khả năng mở rộng và phân tán

Nginx dễ dàng đóng vai trò:

- Load Balancer phân phối yêu cầu đến nhiều Apache phía sau.
- Gateway để chia tách các ứng dụng khác nhau (API, frontend).

### 4. Đơn giản hóa cấu hình đa miền

Nginx hỗ trợ Virtual Host mạnh mẽ:

- Quản lý nhiều site trên một máy chủ.
- Kết hợp cả HTTP và HTTPS dễ dàng.
- Gắn SSL riêng biệt cho từng domain.

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

## VI. Kết luận

Mô hình Reverse Proxy với Nginx đứng trước Apache trong môi trường LEMP mang lại:

- Tối ưu hiệu suất và bảo mật.
- Dễ cấu hình nhiều website và chứng chỉ SSL.
- Dễ mở rộng và quản lý trong môi trường thực tế.

Phù hợp cho hệ thống nhỏ đến trung bình cần hiệu suất cao và bảo mật tốt.
