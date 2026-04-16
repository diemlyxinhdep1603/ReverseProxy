

```markdown
# GIAI ĐOẠN 1: CHUẨN BỊ MÔI TRƯỜNG & KHỞI TẠO DỮ LIỆU

Giai đoạn này tập trung vào việc đưa mã nguồn lên máy chủ, thiết lập cơ sở dữ liệu và phân quyền hệ thống. Đây là nền móng quan trọng để đảm bảo Web Server có thể đọc và thực thi mã nguồn chính xác mà không gặp lỗi phân quyền.

## 1. Truyền tải dữ liệu lên VPS (Sử dụng SCP)

Sử dụng lệnh `scp` để đẩy các thư mục mã nguồn và file database từ máy cá nhân lên thư mục tạm `/root/` trên VPS.

```bash
# Thực hiện trên Terminal của máy cá nhân (Mac/Linux)
# Đẩy mã nguồn Laravel và WordPress
scp -r /Users/lyhuynh/Downloads/laravel_source root@221.132.21.142:/root/
scp -r /Users/lyhuynh/Downloads/source_wq root@221.132.21.142:/root/

# Đẩy các file cơ sở dữ liệu (.sql)
scp /Users/lyhuynh/Downloads/linhlt_db.sql root@221.132.21.142:/root/
scp /Users/lyhuynh/Downloads/linhlt_wp_lodoz.sql root@221.132.21.142:/root/
```

## 2. Thiết lập cấu trúc thư mục hệ thống

Sau khi dữ liệu đã được tải lên thành công, chúng ta tiến hành kết nối SSH và di chuyển các thư mục về đường dẫn tiêu chuẩn `/var/www/`.

```bash
# Đăng nhập vào VPS
ssh root@221.132.21.142

# Di chuyển dữ liệu về thư mục quản lý tập trung
mv /root/laravel_source /var/www/
mv /root/source_wq /var/www/
mv /root/*.sql /var/www/
```

## 3. Khởi tạo Cơ sở dữ liệu (Import SQL)

Thực hiện nạp dữ liệu từ các file `.sql` vào những cơ sở dữ liệu rỗng đã được chuẩn bị sẵn.

```bash
cd /var/www/

# Import dữ liệu cho dự án Laravel
mysql -u root -p laravel_db < linhlt_db.sql

# Import dữ liệu cho dự án WordPress
mysql -u root -p wordpress_db < linhlt_wp_lodoz.sql
```
> **Lưu ý:** Nhập mật khẩu `root` của MySQL khi được yêu cầu. Đảm bảo các cơ sở dữ liệu `laravel_db` và `wordpress_db` đã được tạo trước đó.

## 4. Phân quyền hệ thống (Ownership & Permissions)

Thao tác này đảm bảo dịch vụ Web (User `www-data`) có đủ quyền để đọc code và ghi logs.

### 4.1 Cấp quyền sở hữu (CHOWN)
Chuyển chủ sở hữu từ `root` sang `www-data` để Nginx và Apache có thể truy cập mã nguồn.

```bash
chown -R www-data:www-data /var/www/laravel_source
chown -R www-data:www-data /var/www/source_wq
```

### 4.2 Cấp quyền đặc biệt cho Laravel (CHMOD)
Framework Laravel yêu cầu quyền ghi (Write) vào các thư mục `storage` và `cache` để vận hành hệ thống.

```bash
chmod -R 775 /var/www/laravel_source/storage
chmod -R 775 /var/www/laravel_source/bootstrap/cache
```

## 5. Dọn dẹp và Bảo mật (Cleanup)

Xóa bỏ các file `.sql` ngay sau khi Import để tránh rò rỉ thông tin nhạy cảm của cơ sở dữ liệu.

```bash
rm /var/www/linhlt_db.sql /var/www/linhlt_wp_lodoz.sql
```

---

### 📖 Giải thích kỹ thuật:
* **Quyền 775:** Cho phép Chủ sở hữu (Owner) và Nhóm (Group www-data) có quyền Đọc/Ghi/Chạy, trong khi người dùng khác chỉ có quyền Đọc/Chạy. Điều này đảm bảo tính bảo mật tối thiểu cho máy chủ.
* **Logic di chuyển file:** Việc đưa code về `/var/www/` giúp hệ thống tuân thủ cấu trúc phân cấp thư mục của Linux (FHS), giúp các kỹ sư hệ thống khác dễ dàng quản trị và bảo trì sau này.
```
