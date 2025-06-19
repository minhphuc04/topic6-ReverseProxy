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

| Website                 | Loại        | Domain                          |
|-------------------------|-------------|----------------------------------|
| Website WordPress       | CMS         | `mphuc.wp.vietnix.tech`         |
| Website Laravel         | Framework   | `mphuc.laravel.vietnix.tech`    |

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

# IV. Giải thích: Vì sao Nginx đứng trước Apache?

Việc triển khai mô hình Reverse Proxy với Nginx đứng trước Apache là một lựa chọn kiến trúc phổ biến và có lý do rõ ràng về mặt hiệu năng, bảo mật và khả năng mở rộng.

## 1. Hiệu năng cao hơn

Nginx được thiết kế theo mô hình **event-driven, non-blocking I/O**, giúp xử lý hàng ngàn kết nối đồng thời với mức tài nguyên thấp hơn so với Apache (sử dụng mô hình process hoặc thread).

Khi đứng trước Apache, Nginx xử lý các kết nối tĩnh (static content) như CSS, JS, ảnh, video cực kỳ hiệu quả và chuyển tiếp (proxy) các yêu cầu động (PHP) về Apache xử lý.

Điều này giúp giảm tải cho Apache, tối ưu tài nguyên và thời gian phản hồi.

## 2. Tăng cường bảo mật

Nginx đóng vai trò như một tường chắn đầu tiên (frontend), giúp:

- Chặn các truy cập không hợp lệ, spam, DDoS cơ bản.
- Ẩn chi tiết nội bộ của Apache khỏi client.
- Kiểm soát SSL termination tập trung.

Chứng chỉ SSL (ZeroSSL) được xử lý tại Nginx, giúp đơn giản hóa cấu hình SSL cho Apache phía sau.

## 3. Khả năng mở rộng và phân tán tốt hơn

Khi cần mở rộng quy mô, Nginx dễ dàng đóng vai trò:

- Load Balancer phân phối yêu cầu đến nhiều Apache backend.
- Gateway phân tách dịch vụ hoặc API riêng biệt.

Nhờ đó, kiến trúc hệ thống có thể phân tầng rõ ràng, dễ giám sát, dễ nâng cấp.

## 4. Đơn giản hóa cấu hình đa miền (multi-domain)

Với khả năng cấu hình Virtual Host linh hoạt, Nginx dễ dàng quản lý:

- Nhiều website (WordPress, Laravel, phpMyAdmin, mặc định)
- Hỗ trợ cả HTTP và HTTPS đồng thời
- Gắn chứng chỉ riêng cho từng miền

# V. Kết quả đạt được

Sau khi triển khai đầy đủ theo yêu cầu, hệ thống đạt được các tiêu chí sau:

###  1. Mô hình hoạt động hoàn chỉnh

- Nginx hoạt động như reverse proxy cho toàn bộ hệ thống
- Apache xử lý các truy vấn động từ WordPress và Laravel
- Tách biệt rõ các nhiệm vụ giữa 2 webserver

###  2. Hai website hoạt động độc lập

| Domain                                | Trạng thái | Ghi chú                         |
|---------------------------------------|------------|----------------------------------|
| https://mphuc.wp.vietnix.tech         |  Online  | WordPress qua Nginx + Apache    |
| https://mphuc.laravel.vietnix.tech    |  Online  | Laravel qua Nginx + Apache      |

###  3. Hỗ trợ đầy đủ cả HTTP và HTTPS

- Tự động redirect từ HTTP sang HTTPS (nếu cấu hình)
- Chứng chỉ từ ZeroSSL hoạt động bình thường, được client tin cậy

###  4. Default vhost xử lý đúng

- Mọi domain lạ hoặc truy cập bằng IP đều bị chặn hoặc trả về thông báo "Access Denied"
- Tránh rò rỉ thông tin hệ thống

###  5. phpMyAdmin truy cập nội bộ

- phpMyAdmin cài thành công
- Có thể cấu hình truy cập nội bộ qua domain tùy chọn

# VI. Kết luận

Việc triển khai mô hình Reverse Proxy Nginx–Apache trong môi trường LEMP giúp:

- Tăng cường hiệu suất và bảo mật hệ thống
- Dễ dàng mở rộng và cấu hình nhiều domain với SSL
- Quản lý linh hoạt các dịch vụ web (WordPress, Laravel, phpMyAdmin)
