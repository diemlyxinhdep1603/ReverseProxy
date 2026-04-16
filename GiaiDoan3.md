

# GIAI ĐOẠN 3: CẤU HÌNH APACHE VIRTUAL HOST (XỬ LÝ LOGIC)

Sau khi đã mở các cổng phụ (8080 và 8443) ở Giai đoạn 2, Apache hiện giống như một tòa nhà đã mở cửa nhưng chưa chia phòng ban. Giai đoạn này chúng ta sẽ thiết lập **Virtual Host (VHost)** để định danh từng website.

## 1. Mục tiêu của Giai đoạn 3
* **Phân chia khu vực:** Tách biệt mã nguồn WordPress (`source_wq`) và Laravel (`laravel_source`).
* **Định danh tên miền:** Gán đúng domain `wp.diemly...` và `laravel.diemly...` cho từng thư mục code tương ứng.
* **Kích hoạt thực thi .htaccess:** Đây là điểm mấu chốt để các đường dẫn trên website không bị lỗi **404 Not Found**.

---

## 2. Các bước triển khai chi tiết

### Bước 1: Thiết lập Virtual Host cho WordPress
Chúng ta tạo file cấu hình để Apache biết cách đón khách cho trang WordPress.

```bash
sudo nano /etc/apache2/sites-available/wp.diemly.vietnix.tech.conf
```

**Nội dung cấu hình:**
```apache
# --- LUỒNG HTTP (Cổng 8080) ---
<VirtualHost *:8080>
    ServerName wp.diemly.vietnix.tech
    DocumentRoot /var/www/source_wq
    
    <Directory /var/www/source_wq>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>

# --- LUỒNG HTTPS (Cổng 8443) ---
<VirtualHost *:8443>
    ServerName wp.diemly.vietnix.tech
    DocumentRoot /var/www/source_wq
    
    # Kích hoạt SSL nội bộ để khớp luồng HTTPS từ Nginx
    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/wp.diemly.vietnix.tech/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/wp.diemly.vietnix.tech/privkey.pem
    
    <Directory /var/www/source_wq>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### Bước 2: Thiết lập Virtual Host cho Laravel
Lưu ý quan trọng: Laravel yêu cầu trỏ trực tiếp vào thư mục `/public`.

```bash
sudo nano /etc/apache2/sites-available/laravel.diemly.vietnix.tech.conf
```

**Nội dung cấu hình:**
```apache
# --- LUỒNG HTTP (Cổng 8080) ---
<VirtualHost *:8080>
    ServerName laravel.diemly.vietnix.tech
    DocumentRoot /var/www/laravel_source/public
    
    <Directory /var/www/laravel_source/public>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>

# --- LUỒNG HTTPS (Cổng 8443) ---
<VirtualHost *:8443>
    ServerName laravel.diemly.vietnix.tech
    DocumentRoot /var/www/laravel_source/public
    
    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/laravel.diemly.vietnix.tech/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/laravel.diemly.vietnix.tech/privkey.pem
    
    <Directory /var/www/laravel_source/public>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

---

## 3. Giải trình mã lệnh (Dành cho Leader)

* **`<VirtualHost *:8080>` và `<VirtualHost *:8443>`**: Đáp ứng yêu cầu "HTTP đi với HTTP, HTTPS đi với HTTPS". Việc tách biệt giúp Apache phân loại luồng dữ liệu và sử dụng đúng khóa SSL (`SSLEngine on`) cho luồng 8443.
* **DocumentRoot (Laravel vs WordPress)**: 
    * WordPress trỏ về thư mục gốc.
    * Laravel trỏ về `/public`: Đây là cơ chế bảo mật bắt buộc của Laravel để bảo vệ các file hệ thống và file `.env` chứa mật khẩu không bị truy cập trái phép từ bên ngoài.
* **AllowOverride All**: Dòng lệnh này cho phép Apache đọc và thực thi file `.htaccess`. Đây là tính năng Nginx không có, giúp xử lý các đường dẫn URL thân thiện một cách tự động.

---

## 4. Kích hoạt và Kiểm tra hệ thống

Sau khi "xây phòng" xong, chúng ta tiến hành mở cửa các căn phòng này và đóng phòng mặc định của hệ thống.

```bash
# 1. Tắt vhost mặc định của Apache để tránh nhiễu
sudo a2dissite 000-default.conf

# 2. Bật 2 vhost vừa tạo
sudo a2ensite wp.diemly.vietnix.tech.conf laravel.diemly.vietnix.tech.conf

# 3. Kiểm tra lỗi cú pháp (Cực kỳ quan trọng)
sudo apache2ctl configtest
```

> **Kết quả mong đợi:** Màn hình trả về **Syntax OK**. Nếu báo lỗi, cần kiểm tra lại đường dẫn chứng chỉ SSL hoặc các dấu đóng mở ngoặc.

Cuối cùng, khởi động lại Apache:
```bash
sudo systemctl restart apache2
```

