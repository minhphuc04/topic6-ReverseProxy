
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
    listen [::]:80 default_server;
    server_name _;

    return 403 "Access Denied. Not allowed.";
}

server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;
    server_name _;

    ssl_certificate     /etc/ssl/mphuc_wp/certificate.crt;
    ssl_certificate_key /etc/ssl/mphuc_wp/private.key;

    return 403 "Access Denied. Not allowed.";
}


```

---

## IV. Cấu hình Nginx Reverse Proxy hoàn chỉnh
### 1. Website WordPress (mphuc.wp.vietnix.tech)
---
server {
    listen 80;
    server_name mphuc.wp.vietnix.tech;

    # Tự động chuyển HTTP sang HTTPS
    return 301 https://$host$request_uri;
}
---

server {
    listen 443 ssl;
    server_name mphuc.wp.vietnix.tech;

    # Đường dẫn chứng chỉ thực tế đã tạo (fullchain)
    ssl_certificate     /etc/ssl/mphuc_wp/fullchain.crt;
    ssl_certificate_key /etc/ssl/mphuc_wp/private.key;

    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    # Giao tiếp về Apache (chạy port 8081 hoặc 8080 tùy bạn cấu hình Apache vhost)
    location / {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Phục vụ tài nguyên tĩnh trực tiếp từ Nginx (nếu cần)
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg|eot|mp4|webp)$ {
        root /var/www/wordpress;
        expires 30d;
        access_log off;
        try_files $uri $uri/ =404;
    }
}

### 2. Website Laravel (mphuc.laravel.vietnix.tech)

server {
    listen 80;
    server_name mphuc.laravel.vietnix.tech;

    # Tự động chuyển HTTP sang HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name mphuc.laravel.vietnix.tech;

    ssl_certificate     /etc/ssl/mphuc_laravel/fullchain.crt;
    ssl_certificate_key /etc/ssl/mphuc_laravel/private.key;

    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    # Giao tiếp về Apache (chạy port 8082 hoặc 8080 tùy bạn cấu hình Apache vhost)
    location / {
        proxy_pass http://127.0.0.1:8082;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Phục vụ tài nguyên tĩnh trực tiếp từ Nginx (nếu cần)
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg|eot|mp4|webp)$ {
        root /var/www/laravel/public;
        expires 30d;
        access_log off;
        try_files $uri $uri/ =404;
    }
}

    Lưu ý: Nếu cả hai site Apache đều dùng chung port 8080, hãy đảm bảo vhost Apache tách biệt theo ServerName. Nếu mỗi site một port, chỉnh lại proxy_pass cho đúng port backend.

### 3. Default Vhost (Xử lý IP/domain lạ)

server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;

    return 403 "Access Denied. Not allowed.";
}

server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;
    server_name _;

    ssl_certificate     /etc/ssl/mphuc_wp/certificate.crt;
    ssl_certificate_key /etc/ssl/mphuc_wp/private.key;

    return 403 "Access Denied. Not allowed.";
}


## V. Giải thích: Vì sao Nginx đứng trước Apache?

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

## *. Tận dụng điểm mạnh của NGINX khi đứng trước Apache

### 1. Mô hình Reverse Proxy: NGINX trước – Apache sau

Khi triển khai NGINX đứng trước Apache theo mô hình Reverse Proxy, hệ thống được chia nhiệm vụ như sau:

| Thành phần | Nhiệm vụ chính |
|------------|----------------|
| NGINX      | Xử lý các request tĩnh (ảnh, JS, CSS), thực hiện SSL termination, routing/phân luồng, caching. |
| Apache     | Xử lý các request động (PHP, Laravel, WordPress), xử lý logic ứng dụng và truy vấn CSDL. |

### 2. Mục tiêu của việc tận dụng NGINX

- Tăng hiệu năng phục vụ tài nguyên tĩnh.
- Giảm tải cho Apache – vốn chỉ nên tập trung xử lý logic động.
- Tối ưu bảo mật, tốc độ phản hồi, tài nguyên hệ thống.
- Dễ mở rộng và bảo trì hệ thống.

### 3. Cách tận dụng điểm mạnh của NGINX

### 3.1. Phục vụ tài nguyên tĩnh trực tiếp từ NGINX

Tài nguyên như `.css`, `.js`, `.jpg`, `.png`, `.svg`, `.woff` nên được xử lý trực tiếp bởi NGINX, không chuyển tiếp qua Apache, giúp tăng hiệu suất và giảm tải cho backend.

**Ví dụ cấu hình cho từng website:**

#### Với WordPress:

```nginx
location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg|eot|mp4|webp)$ {
    root /var/www/wordpress;
    expires 30d;
    access_log off;
    try_files $uri $uri/ =404;
}
```

#### Với Laravel:

```nginx
location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg|eot|mp4|webp)$ {
    root /var/www/laravel/public;
    expires 30d;
    access_log off;
    try_files $uri $uri/ =404;
}
```

**Giải thích:**
- `root`: trỏ về đúng thư mục chứa mã nguồn tĩnh của từng website.
- `expires 30d`: cho phép trình duyệt cache 30 ngày.
- `access_log off`: giảm ghi log không cần thiết.
- `try_files`: bảo vệ chống lỗi truy cập file không tồn tại.

---

### 3.2. Caching nội dung bằng NGINX (Proxy Cache)

Caching giúp tăng tốc cho các request lặp lại (đặc biệt với file HTML hoặc JSON không đổi trong thời gian ngắn), giảm số request xuống Apache.

**Ví dụ cấu hình caching cho từng website:**

#### WordPress (proxy_pass về Apache port 8081):

```nginx
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=wp_cache:10m inactive=60m;
proxy_cache_key "$scheme$request_method$host$request_uri";

location / {
    proxy_pass http://127.0.0.1:8081;
    proxy_cache wp_cache;
    proxy_cache_valid 200 302 10m;
    proxy_cache_valid 404 1m;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

#### Laravel (proxy_pass về Apache port 8082):

```nginx
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=laravel_cache:10m inactive=60m;
proxy_cache_key "$scheme$request_method$host$request_uri";

location / {
    proxy_pass http://127.0.0.1:8082;
    proxy_cache laravel_cache;
    proxy_cache_valid 200 302 10m;
    proxy_cache_valid 404 1m;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

**Giải thích:**
- `proxy_cache_path`: tạo bộ nhớ cache riêng cho từng site.
- `proxy_cache_key`: định nghĩa khóa cache.
- `proxy_pass`: chuyển tiếp request về đúng Apache backend theo từng website.
- `proxy_cache_valid`: chỉ định thời gian hợp lệ của cache cho các trạng thái HTTP.
- Các dòng `proxy_set_header` đảm bảo Apache nhận đúng thông tin header từ client.

---

### 3.3. Tách thư mục chứa file tĩnh riêng biệt

Thay vì để Laravel hoặc WordPress sinh file tĩnh trong cùng thư mục web gốc, có thể tách file tĩnh riêng tại `/static` để quản lý dễ hơn (tùy chọn nâng cao).

**Ví dụ cấu hình cho từng website:**

#### WordPress

```nginx
location /static/ {
    root /var/www/wordpress;
    expires 30d;
}
```

#### Laravel

```nginx
location /static/ {
    root /var/www/laravel/public;
    expires 30d;
}
```

**Giải thích:**
- Khi dùng cấu hình này, các file tĩnh nên để hoặc liên kết (symlink) về thư mục `/static` trong code.
- Các URL ảnh/CSS/JS trong website cần trỏ đúng về `/static/...` để NGINX xử lý trực tiếp.

---

### 3.4. SSL Termination tại NGINX

Để giảm tải mã hóa SSL cho Apache, NGINX nên đứng trước để nhận HTTPS, sau đó proxy về Apache bằng HTTP nội bộ. Cấu hình dưới đây sử dụng đúng chứng chỉ bạn đã gộp (fullchain) và private key.

#### WordPress (`mphuc.wp.vietnix.tech`):

```nginx
server {
    listen 443 ssl;
    server_name mphuc.wp.vietnix.tech;

    ssl_certificate     /etc/ssl/mphuc_wp/fullchain.crt;
    ssl_certificate_key /etc/ssl/mphuc_wp/private.key;

    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

#### Laravel (`mphuc.laravel.vietnix.tech`):

```nginx
server {
    listen 443 ssl;
    server_name mphuc.laravel.vietnix.tech;

    ssl_certificate     /etc/ssl/mphuc_laravel/fullchain.crt;
    ssl_certificate_key /etc/ssl/mphuc_laravel/private.key;

    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://127.0.0.1:8082;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Giải thích:**
- `listen 443 ssl`: server lắng nghe cổng HTTPS.
- `ssl_certificate`, `ssl_certificate_key`: đường dẫn file chứng chỉ thực tế của bạn (đã ghép fullchain).
- `proxy_pass`: chuyển tiếp request tới Apache backend đúng port từng website.
- Các `proxy_set_header` đảm bảo Apache backend nhận đúng thông tin từ client.

---

### 4. Luồng xử lý thực tế

- Client gửi request HTTP/HTTPS đến NGINX (port 80/443).
- NGINX:
  - Nếu là tài nguyên tĩnh: xử lý trực tiếp, không cần Apache.
  - Nếu là nội dung động: proxy về Apache theo port nội bộ.
- Apache xử lý request động (Laravel, WordPress...), trả lại nội dung cho NGINX → gửi cho client.

### 5. Ưu điểm nổi bật

| Ưu điểm           | Mô tả |
|-------------------|-------|
| Tối ưu hiệu năng  | NGINX dùng mô hình event-driven nên phục vụ file tĩnh cực nhanh. |
| Giảm tải Apache   | Apache không phải phục vụ file tĩnh → RAM/CPU chỉ tập trung xử lý PHP. |
| Tăng bảo mật      | NGINX ẩn server backend (Apache), ngăn dò server thực. |
| Dễ mở rộng        | NGINX dễ load balancing, mở rộng thêm backend sau này. |

## VII. Kết quả đạt được

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

## VIII. Kết luận

Mô hình Reverse Proxy với Nginx đứng trước Apache trong môi trường LEMP mang lại:

- Tối ưu hiệu suất và bảo mật.
- Dễ cấu hình nhiều website và chứng chỉ SSL.
- Dễ mở rộng và quản lý trong môi trường thực tế.

Phù hợp cho hệ thống nhỏ đến trung bình cần hiệu suất cao và bảo mật tốt.

---

## IX. So sánh chi tiết: Nginx và Apache

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

## XX. Tại sao kết hợp cả hai?

| Kết hợp                 | Lợi ích                                                  |
|-------------------------|----------------------------------------------------------|
| `Nginx (proxy)`         | Xử lý SSL, tĩnh, redirect, load balancing                |
| `Apache (backend)`      | Xử lý nội dung động, PHP, CMS Laravel/WordPress         |
| Nginx → Apache          | Giao tiếp ngược proxy_pass để tận dụng sức mạnh riêng   |

=> **Tận dụng điểm mạnh của cả hai**, xây dựng hệ thống bảo mật – hiệu năng cao – dễ mở rộng.
