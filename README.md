
# Triển khai mô hình Reverse Proxy kết hợp giữa Nginx và Apache từ LEMP

## I. Giới thiệu

Tiếp tục từ mô hình **LEMP** (Linux – Nginx – MySQL – PHP) đã cài đặt trước đó, bài này xây dựng mô hình **Reverse Proxy** kết hợp **Nginx** và **Apache** để chạy 2 website với yêu cầu như sau:
## II. Xây dựng mô hình reverse proxy kết hợp nginx và apache
### Bước 1: Cài các gói cần thiết
sudo apt update && sudo apt install apache2 php libapache2-mod-php php-mysql mysql-server phpmyadmin unzip -y
### Bước 2: Apache listen cổng 8080
Listen 8080
Listen 8443
### Bước 3:. Cấu hình Nginx Reverse Proxy hoàn chỉnh
### 1. Website WordPress (mphuc.wp.vietnix.tech)
### Chỉnh sửa file Nginx (/etc/nginx/sites-available/mphuc.wp.vietnix.tech)
---
```server {
    listen 80;
    server_name mphuc.wp.vietnix.tech;
    
    location ~* \.(gif|jpg|jpeg|png|ico|wmv|3gp|avi|mpg|mpeg|mp4|flv|mp3|mid|js|css|html|htm|wml)$ {
        root /var/www/html/Source_wp;
        expires 30d;
    }

    if ($http_user_agent ~* (wget|curl|sqlmap|nessus|acunetix|fimap|nikto|scanner)) {
        return 403;
    }

    if ($request_method !~ ^(GET|POST|HEAD)$) {
        return 444;
    }

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 443 ssl http2;
    server_name mphuc.wp.vietnix.tech;

    ssl_certificate /etc/ssl/mphuc_wp/certificate.crt;
    ssl_certificate_key /etc/ssl/mphuc_wp/private.key;
    ssl_trusted_certificate /etc/ssl/mphuc_wp/ca_bundle.crt;

    location ~* \.(gif|jpg|jpeg|png|ico|wmv|3gp|avi|mpg|mpeg|mp4|flv|mp3|mid|js|css|html|htm|wml)$ {
        root /var/www/html/Source_wp;
        expires 30d;
    }

    if ($http_user_agent ~* (wget|curl|sqlmap|nessus|acunetix|fimap|nikto|scanner)) {
        return 403;
    }

    if ($request_method !~ ^(GET|POST|HEAD)$) {
        return 444;
    }

    location / {
        proxy_pass https://127.0.0.1:8443;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
---
### Chỉnh sửa file Apache (/etc/apache2/sites-available/mphuc_wp.conf)
---
```
<VirtualHost *:8080>
    DocumentRoot /var/www/html/Source_wp
    ServerName mphuc.wp.vietnix.tech

    <Directory "/var/www/html/Source_wp">
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/mphuc.wp.vietnix.tech-error.log
    CustomLog ${APACHE_LOG_DIR}/mphuc.wp.vietnix.tech-access.log combined
</VirtualHost>

<VirtualHost *:8443>
    DocumentRoot /var/www/html/Source_wp
    ServerName mphuc.wp.vietnix.tech

    SSLEngine on
    SSLCertificateFile /etc/ssl/mphuc_wp/certificate.crt
    SSLCertificateKeyFile /etc/ssl/mphuc_wp/private.key
    SSLCertificateChainFile /etc/ssl/mphuc_wp/ca_bundle.crt

    <Directory "/var/www/html/Source_wp">
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/mphuc.wp.vietnix.tech-error.log
    CustomLog ${APACHE_LOG_DIR}/mphuc.wp.vietnix.tech-access.log combined
</VirtualHost>
```
---
### 2. Website Laravel (mphuc.laravel.vietnix.tech)
### Chỉnh sửa file Nginx (/etc/nginx/sites-available/mphuc.laravel.vietnix.tech)
```
server {
    listen 80;
    server_name mphuc.laravel.vietnix.tech;

    location ~* \.(gif|jpg|jpeg|png|ico|wmv|3gp|avi|mpg|mpeg|mp4|flv|mp3|mid|js|css|html|htm|wml)$ {
        root /var/www/laravel/public;
        expires 30d;
    }

    if ($http_user_agent ~* (wget|curl|sqlmap|nessus|acunetix|fimap|nikto|scanner)) {
        return 403;
    }

    if ($request_method !~ ^(GET|POST|HEAD)$) {
        return 444;
    }

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 443 ssl http2;
    server_name mphuc.laravel.vietnix.tech;

    ssl_certificate /etc/ssl/mphuc_laravel/certificate.crt;
    ssl_certificate_key /etc/ssl/mphuc_laravel/private.key;
    ssl_trusted_certificate /etc/ssl/mphuc_laravel/ca_bundle.crt;

    location ~* \.(gif|jpg|jpeg|png|ico|wmv|3gp|avi|mpg|mpeg|mp4|flv|mp3|mid|js|css|html|htm|wml)$ {
        root /var/www/laravel/public;
        expires 30d;
    }

    if ($http_user_agent ~* (wget|curl|sqlmap|nessus|acunetix|fimap|nikto|scanner)) {
        return 403;
    }

    if ($request_method !~ ^(GET|POST|HEAD)$) {
        return 444;
    }

    location / {
        proxy_pass https://127.0.0.1:8443;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
### Chỉnh sửa file Apache (/etc/apache2/sites-available/mphuc_laravel.conf)
```
<VirtualHost *:8080>
    DocumentRoot /var/www/laravel/public
    ServerName mphuc.laravel.vietnix.tech

    <Directory "/var/www/laravel/public">
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/mphuc.laravel.vietnix.tech-error.log
    CustomLog ${APACHE_LOG_DIR}/mphuc.laravel.vietnix.tech-access.log combined

    RewriteEngine On
</VirtualHost>

<VirtualHost *:8443>
    DocumentRoot /var/www/laravel/public
    ServerName mphuc.laravel.vietnix.tech

    SSLEngine on
    SSLCertificateFile /etc/ssl/mphuc_laravel/certificate.crt
    SSLCertificateKeyFile /etc/ssl/mphuc_laravel/private.key
    SSLCertificateChainFile /etc/ssl/mphuc_laravel/ca_bundle.crt

    <Directory "/var/www/laravel/public">
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/mphuc.laravel.vietnix.tech-error.log
    CustomLog ${APACHE_LOG_DIR}/mphuc.laravel.vietnix.tech-access.log combined

    RewriteEngine On
</VirtualHost>
```
### 3. Default Vhost (Xử lý IP/domain lạ)
```
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

### Bước 4: Sửa lại nội dung file TrustProxies.php để Laravel tin tưởng proxy (TrustProxies Middleware)

nano /var/www/mphuc_laravel/app/Http/Middleware/TrustProxies.php

Chỉnh sửa dòng protected $proxies; thành protected $proxies='*';
Sau đó kích hoạt:
sudo ln -s /etc/nginx/sites-available/mphuc.laravel.vietnix.tech /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/mphuc.wp.vietnix.tech /etc/nginx/sites-enabled/
### Bước 5: Khởi động lại dịch vụ
sudo systemctl restart apache2
sudo systemctl restart nginx
### Bước 6: Bật module SSL:
sudo a2enmod ssl
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

    ssl_certificate /etc/ssl/mphuc_wp/certificate.crt;
    ssl_certificate_key /etc/ssl/mphuc_wp/private.key;
    ssl_trusted_certificate /etc/ssl/mphuc_wp/ca_bundle.crt;

#### Laravel (`mphuc.laravel.vietnix.tech`):

    ssl_certificate /etc/ssl/mphuc_laravel/certificate.crt;
    ssl_certificate_key /etc/ssl/mphuc_laravel/private.key;
    ssl_trusted_certificate /etc/ssl/mphuc_laravel/ca_bundle.crt;


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



## V. So sánh chi tiết: Nginx và Apache

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

## VI. Tại sao kết hợp cả hai?

| Kết hợp                 | Lợi ích                                                  |
|-------------------------|----------------------------------------------------------|
| `Nginx (proxy)`         | Xử lý SSL, tĩnh, redirect, load balancing                |
| `Apache (backend)`      | Xử lý nội dung động, PHP, CMS Laravel/WordPress         |
| Nginx → Apache          | Giao tiếp ngược proxy_pass để tận dụng sức mạnh riêng   |

=> **Tận dụng điểm mạnh của cả hai**, xây dựng hệ thống bảo mật – hiệu năng cao – dễ mở rộng.
