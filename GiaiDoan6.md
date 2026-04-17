
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
* `$table_prefix`: **'Sa3QIZ_'** (Đây là điểm mấu chốt, phải khớp với tiền tố bảng trong file .sql).

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

## 3. Nhật ký Debug: 

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

## 4. Công cụ kiểm tra

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


## 6. Xử lý lỗi "Hardcoded Links" sau khi Migrate Website

### 6.1. Định nghĩa lỗi "Hardcoded Links" là gì?
Lỗi **Hardcoded Links (Đường dẫn bị viết chết/viết cứng)** thường xảy ra sau khi chuyển dữ liệu (Migrate) một website WordPress từ môi trường cũ (máy Leader) sang môi trường mới (VPS Production).

* **Bản chất vấn đề:** WordPChào bạn, đúng là trong bản bạn vừa gửi, phần **Bước 1** vẫn đang dùng bản cũ bị thiếu mất khâu "mở khóa" (đổi mật khẩu). Không có bước đổi pass thì chúng ta có ép quyền Admin cũng không thể đăng nhập vào được.

Mình đã gộp lại toàn bộ và chỉnh sửa cho hoàn chỉnh nhất. Bạn chỉ cần copy toàn bộ đoạn Markdown dưới đây và thay thế vào file báo cáo của mình nhé:

## 6. Xử lý lỗi "Hardcoded Links" sau khi Migrate Website

### 6.1. Định nghĩa lỗi "Hardcoded Links" là gì?
Lỗi **Hardcoded Links (Đường dẫn bị viết cứng)** thường xảy ra sau khi chuyển dữ liệu (Migrate) một website WordPress từ môi trường cũ (máy Leader) sang môi trường mới (VPS Production).

* **Bản chất vấn đề:** WordPress và các Plugin (Trình dựng trang, menu, slider...) lưu trực tiếp tên miền cũ (`linhlt.id.vn`) vào thẳng cơ sở dữ liệu (Database) dưới dạng văn bản tĩnh.
* **Biểu hiện:** Mặc dù đã cấu hình URL mới trong file `wp-config.php`, nhưng khi người dùng click vào logo, hình ảnh, hoặc các liên kết trên menu, trình duyệt vẫn tự động điều hướng sang trang web cũ hoặc báo lỗi bảo mật (SSL không hợp lệ).

### 6.2. Phương pháp tiếp cận và giải quyết
Để xử lý triệt để, hệ thống yêu cầu can thiệp trực tiếp vào Database. Tuy nhiên, **không được sử dụng lệnh SQL `UPDATE` thông thường** cho toàn bộ dữ liệu. 

* **Lý do:** WordPress lưu trữ nhiều cấu hình dưới định dạng **Serialized Data (Dữ liệu chuỗi hóa)**. Việc dùng lệnh `UPDATE` thuần túy để thay đổi số lượng ký tự của tên miền sẽ làm hỏng (corrupt) cấu trúc mảng dữ liệu, dẫn đến lỗi trắng trang (Fatal Error).
* **Hướng giải quyết:** Bắt buộc sử dụng Plugin chuyên dụng có khả năng đọc/ghi Serialized Data để tìm và thay thế (Search & Replace) tên miền an toàn.

---

### 6.3. Quy trình thực hành chi tiết

Quy trình xử lý chuẩn bao gồm 4 bước kỹ thuật sau:

#### Bước 1: Chiếm quyền và Nâng quyền Administrator qua dòng lệnh SQL (Privilege Escalation)
Để cài đặt công cụ sửa lỗi, tài khoản đăng nhập cần có quyền Administrator. Vì không có mật khẩu của Leader, ta thực hiện đổi mật khẩu và ép quyền trực tiếp qua giao diện dòng lệnh (Terminal):

1. Truy cập MySQL trên VPS: `mysql -u root -p`
2. Truy vấn ID của user hiện tại để xác định mục tiêu (Giả sử tìm được user `admin` có ID là 5):
   ```sql
   SELECT ID, user_login FROM Sa3QIZ_users; 
   ```
3. Đổi mật khẩu cho user ID 5 thành mật khẩu mới (VD: `Ly@2026`) bằng hàm mã hóa MD5:
   ```sql
   UPDATE Sa3QIZ_users SET user_pass = MD5('Ly@2026') WHERE ID = 5;
   ```
4. Cấp quyền Administrator và mức độ ưu tiên cao nhất cho ID 5:
   ```sql
   UPDATE Sa3QIZ_usermeta SET meta_value = 'a:1:{s:13:"administrator";b:1;}' WHERE user_id = 5 AND meta_key = 'Sa3QIZ_capabilities';
   UPDATE Sa3QIZ_usermeta SET meta_value = '10' WHERE user_id = 5 AND meta_key = 'Sa3QIZ_user_level';
   ```
* **Mục đích:** Đặt lại mật khẩu để đăng nhập thành công và khôi phục toàn quyền quản trị, cho phép truy cập menu **Giao diện (Appearance)** và **Gói mở rộng (Plugins)** trên trang Dashboard.

#### Bước 2: Cài đặt công cụ Better Search Replace
1. Đăng nhập trang quản trị WordPress.
2. Điều hướng tới **Plugin** -> **Cài mới (Add New)**.
3. Tìm kiếm, Cài đặt và **Kích hoạt** plugin **Better Search Replace**.
* **Mục đích:** Sử dụng công cụ chuyên dụng để tháo gỡ chuỗi Serialized Data, thay thế tên miền an toàn và đóng gói lại mà không làm vỡ cấu trúc Database.

#### Bước 3: Thực thi quét và thay thế tên miền (Serializing Search & Replace)
Truy cập **Công cụ (Tools)** -> **Better Search Replace** và thực hiện quét 2 lần để đảm bảo không sót giao thức:

* **Lần 1 (Quét HTTP):**
  * **Search for:** `http://linhlt.id.vn`
  * **Replace with:** `https://wp.diemly.vietnix.tech`
* **Lần 2 (Quét HTTPS - Xử lý triệt để):**
  * **Search for:** `https://linhlt.id.vn`
  * **Replace with:** `https://wp.diemly.vietnix.tech`

**Cấu hình bắt buộc khi chạy:**
1. **Select tables:** Chọn toàn bộ bảng trong danh sách. Link cũ có thể nằm ở bất kỳ bảng nào (cấu hình plugin, bài viết, menu...).
2. Bỏ chọn ô **Run as dry run?** (Chạy thử). Nếu không bỏ chọn, hệ thống chỉ báo cáo mô phỏng mà không thực hiện thay đổi dữ liệu thực tế.

#### Bước 4: Xóa bộ nhớ đệm (Clear Cache)
Ngay cả khi Database đã chuẩn, giao diện vẫn có thể lưu cache cũ. Cần làm mới ở 2 tầng:

1. **Tầng Server (LiteSpeed Cache):** Trên thanh công cụ Admin, chọn biểu tượng LiteSpeed -> Nhấp **Purge All (Xóa tất cả)**. Thao tác này xóa các file snapshot HTML tĩnh mang tên miền cũ trên Server.
2. **Tầng Trình duyệt (Client Cache):** Truy cập trang chủ, nhấn **Ctrl + F5** (Windows) hoặc **Cmd + Shift + R** (Mac) để thực hiện Hard Reload, ép trình duyệt tải lại toàn bộ tài nguyên mới nhất từ VPS.
