
```markdown
# GIAI ĐOẠN 6: DEBUG & KẾT NỐI CƠ SỞ DỮ LIỆU 

Sau khi xây dựng xong hệ thống Web Server, bước cuối cùng và quan trọng nhất là "cắm điện" cho ứng dụng bằng cách kết nối mã nguồn với cơ sở dữ liệu và xử lý các lỗi phát sinh do sự khác biệt môi trường.

---

## 1. Kết nối Database cho WordPress (Fix lỗi Connection)

Mã nguồn của Leader mặc định cấu hình theo máy cá nhân, chúng ta cần cập nhật lại thông tin để khớp với Database trên VPS.

**Thao tác:**
```bash
sudo nano /var/www/source_wq/wp-config.php
```

**Các thông số đã fix để chạy đúng:**
* `DB_NAME`: 'wordpress_db'
* `DB_USER`: 'wp_ly' (User bạn đã tạo)
* `DB_PASSWORD`: 'Ly@Vietnix2026!Wp'
* `$table_prefix`: **'Sa3QIZ_'** (Đây là điểm mấu chốt, phải khớp với tiền tố bảng trong file .sql của Leader).

**Ép tên miền mới (Fix lỗi nhảy sang trang linhlt.id.vn):**
Thêm 2 dòng này vào trên dòng `/* That's all, stop editing! */`:
```php
define( 'WP_HOME', '[https://wp.diemly.vietnix.tech](https://wp.diemly.vietnix.tech)' );
define( 'WP_SITEURL', '[https://wp.diemly.vietnix.tech](https://wp.diemly.vietnix.tech)' );
```

---

## 2. Kết nối Database cho Laravel (Fix lỗi 500)

Laravel sử dụng file `.env` để quản lý cấu hình. Chúng ta cần khởi tạo file này từ file mẫu.

**Thao tác:**
```bash
# 1. Tạo file .env từ file mẫu
cp /var/www/laravel_source/.env.example /var/www/laravel_source/.env

# 2. Cấp quyền cho user www-data
chown www-data:www-data /var/www/laravel_source/.env

# 3. Chỉnh sửa cấu hình Database
sudo nano /var/www/laravel_source/.env
```

**Cấu hình chuẩn:**
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=laravel_ly
DB_PASSWORD=Ly@Laravel2026!Sec
```

---

## 3. Nhật ký Debug: Xử lý các lỗi nghiêm trọng

Trong quá trình triển khai, chúng ta đã đối mặt và xử lý thành công các lỗi sau:

### 3.1 Lỗi trắng trang / Critical Error (Xung đột PHP 8.x)
* **Hiện tượng:** Màn hình báo "Đã có một lỗi nghiêm trọng".
* **Nguyên nhân:** Mã nguồn cũ sử dụng cú pháp không còn hỗ trợ trên PHP 8.1.
* **Cách fix:** Bật `WP_DEBUG` để tìm file lỗi, sau đó tắt các Plugin gây xung đột (như YITH Premium) bằng cách đổi tên thư mục plugin.

### 3.2 Lỗi Cache trình duyệt (HSTS)
* **Hiện tượng:** Safari vào được nhưng Chrome báo lỗi "Kết nối không riêng tư" hoặc nhảy về tên miền cũ.
* **Nguyên nhân:** Chrome lưu cache HSTS và Redirect vĩnh viễn của tên miền `linhlt.id.vn`.
* **Cách fix:** Sử dụng tính năng **Empty Cache and Hard Reload** trên Chrome hoặc kiểm tra bằng Tab ẩn danh/Thiết bị mới.

### 3.3 Lỗi Permission (Quyền ghi)
* **Hiện tượng:** Laravel không hiển thị hình ảnh hoặc báo lỗi log.
* **Cách fix:** Đảm bảo thư mục `storage` và `bootstrap/cache` luôn được cấp quyền `775`.

---

## 4. Công cụ kiểm tra (Bảo bối của SysAdmin)

Để chứng minh hệ thống chạy đúng, chúng ta sử dụng lệnh theo dõi log thời gian thực:

* **Xem lỗi Apache:** `sudo tail -f /var/log/apache2/error.log`
* **Xem lỗi Nginx:** `sudo tail -f /var/log/nginx/error.log`
* **Kiểm tra phản hồi Server:** `curl -I https://wp.diemly.vietnix.tech`

---

## 5. Kết quả cuối cùng

Hệ thống đã vận hành hoàn hảo với các chỉ số:
* **Mã phản hồi:** 200 OK.
* **SSL:** Hợp lệ (Green Lock).
* **Giao diện:** Hiển thị đúng nội dung của dự án KIA K5 và Coza Store.
```
