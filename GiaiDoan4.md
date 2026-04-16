

```markdown
# GIAI ĐOẠN 4: CẤU HÌNH NGINX REVERSE PROXY (CHUẨN ENTERPRISE)

Đây là giai đoạn quyết định sự thành công của bài thực hành. Cấu hình này không chỉ đơn thuần là chuyển tiếp yêu cầu, mà còn tối ưu hóa hiệu suất theo tiêu chuẩn doanh nghiệp bằng cách giải quyết triệt để 3 yêu cầu từ Leader:
1. **Tối ưu file tĩnh:** Nginx tự xử lý và trả về các file ảnh, CSS, JS mà không làm phiền Apache.
2. **Luồng dữ liệu song song:** Đảm bảo HTTP đi tới HTTP (8080) và HTTPS đi tới HTTPS (8443).
3. **Không chuyển hướng cưỡng bức:** Loại bỏ lệnh `return 301` để giữ nguyên giao thức khách hàng yêu cầu.

---

## 1. Cấu hình Virtual Host cho WordPress

Chúng ta thiết lập Nginx làm lớp bảo vệ phía trước thư mục `source_wq`.

```bash
sudo nano /etc/nginx/sites-available/wp.diemly.vietnix.tech
```

**Nội dung siêu cấu hình:**

```nginx
# --- KHỐI 1: XỬ LÝ HTTP (CỔNG 80) ➡️ APACHE 8080 ---
server {
    listen 80;
    server_name wp.diemly.vietnix.tech;
    
    root /var/www/source_wq;
    index index.php index.html index.htm;

    # Kiểm tra file tĩnh trước, nếu là PHP hoặc không thấy file thì đẩy sang Apache
    location / {
        try_files $uri $uri/ @apache_http;
    }

    # TỐI ƯU: Nginx tự xử lý các định dạng file tĩnh
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg|eot)$ {
        try_files $uri @apache_http; 
        expires 30d;
        access_log off;
    }

    # Cấu hình Proxy ném sang Apache HTTP
    location @apache_http {
        proxy_pass [http://127.0.0.1:8080](http://127.0.0.1:8080);
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# --- KHỐI 2: XỬ LÝ HTTPS (CỔNG 443) ➡️ APACHE 8443 ---
server {
    listen 443 ssl;
    server_name wp.diemly.vietnix.tech;
    
    root /var/www/source_wq;

    ssl_certificate /etc/letsencrypt/live/wp.diemly.vietnix.tech/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/wp.diemly.vietnix.tech/privkey.pem;

    location / {
        try_files $uri $uri/ @apache_https;
    }

    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg|eot)$ {
        try_files $uri @apache_https;
        expires 30d;
        access_log off;
    }

    # Cấu hình Proxy ném sang Apache HTTPS
    location @apache_https {
        proxy_pass [https://127.0.0.1:8443](https://127.0.0.1:8443);
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_ssl_verify off; # Chấp nhận chứng chỉ nội bộ giữa Proxy
    }
}
```

---

## 2. Cấu hình Virtual Host cho Laravel

Đối với Laravel, đường dẫn `root` phải trỏ chính xác vào thư mục `/public`.

```bash
sudo nano /etc/nginx/sites-available/laravel.diemly.vietnix.tech
```

**Nội dung cấu hình:**

```nginx
# --- KHỐI 1: XỬ LÝ HTTP (CỔNG 80) ➡️ APACHE 8080 ---
server {
    listen 80;
    server_name laravel.diemly.vietnix.tech;
    
    root /var/www/laravel_source/public;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ @apache_http;
    }

    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg|eot)$ {
        try_files $uri @apache_http;
        expires 30d;
        access_log off;
    }

    location @apache_http {
        proxy_pass [http://127.0.0.1:8080](http://127.0.0.1:8080);
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# --- KHỐI 2: XỬ LÝ HTTPS (CỔNG 443) ➡️ APACHE 8443 ---
server {
    listen 443 ssl;
    server_name laravel.diemly.vietnix.tech;
    
    root /var/www/laravel_source/public;

    ssl_certificate /etc/letsencrypt/live/laravel.diemly.vietnix.tech/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/laravel.diemly.vietnix.tech/privkey.pem;

    location / {
        try_files $uri $uri/ @apache_https;
    }

    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg|eot)$ {
        try_files $uri @apache_https;
        expires 30d;
        access_log off;
    }

    location @apache_https {
        proxy_pass [https://127.0.0.1:8443](https://127.0.0.1:8443);
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_ssl_verify off;
    }
}
```

---

## 3. Kiểm tra và Kích hoạt hệ thống

Sau khi hoàn tất chỉnh sửa, chúng ta cần kiểm tra tính chính xác của cú pháp trước khi yêu cầu Nginx nạp lại cấu hình.

```bash
# Kiểm tra lỗi cú pháp
sudo nginx -t
```

Nếu kết quả trả về `syntax is ok` và `test is successful`, thực hiện khởi động lại dịch vụ:

```bash
sudo systemctl restart nginx
```

---

## 4. Giải trình kỹ thuật 

Khi được hỏi về logic xử lý trong file cấu hình này, em sẽ phản hồi dựa trên các điểm cốt lõi sau:
* **Xử lý Static File tại tầng Proxy:** Em đã định nghĩa cụ thể đường dẫn `root` và sử dụng block Regex `location ~* \.(jpg|css...)`. Bên trong đó dùng `try_files $uri` giúp Nginx trực tiếp quét ổ cứng và trả kết quả file tĩnh cho khách hàng. Điều này giúp giảm tải hoàn toàn cho Apache.
* **Luồng dữ liệu End-to-End:** Bằng cách loại bỏ lệnh `return 301`, em đảm bảo luồng đi đúng yêu cầu của anh: Khách vào HTTP sẽ thấy HTTP (qua cổng 8080), khách vào HTTPS sẽ được bảo vệ bởi SSL xuyên suốt (qua cổng 8443).
* **Proxy Headers:** Các dòng `proxy_set_header` giúp truyền thông tin địa chỉ IP thực của khách hàng xuống cho Apache và PHP, đảm bảo các tính năng bảo mật và logs trong code không bị sai lệch (hiển thị IP 127.0.0.1).
```