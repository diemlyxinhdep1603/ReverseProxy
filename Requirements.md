# TRIỂN KHAI HỆ THỐNG REVERSE PROXY NGINX & APACHE
* **Người thực hiện:** Huỳnh Ngọc Diễm Ly
* **Dự án:** Chuyển đổi kiến trúc LEMP sang mô hình Reverse Proxy nâng cao.

---

## 1. Mục tiêu và Yêu cầu chi tiết

### 1.1 Cấu trúc hệ thống (Architecture Setup)
Xây dựng và cấu hình mô hình **Reverse Proxy** kết hợp giữa hai Web Server phổ biến nhất nhằm tối ưu hóa hiệu suất hệ thống:
* **Nginx:**  Đóng vai trò tiếp nhận và xử lý các yêu cầu ban đầu.
* **Apache:**  Đóng vai trò xử lý logic ứng dụng và thực thi PHP.
* **Môi trường:** PHP 8.1, MySQL (MariaDB), và công cụ quản trị phpMyAdmin.
* **Nhiệm vụ lý thuyết:** Tìm hiểu và giải thích lý do kỹ thuật vì sao Nginx thường được đặt đứng trước Apache trong các hệ thống doanh nghiệp.

### 1.2 Triển khai mã nguồn (Web Deployment)
Thực hiện triển khai đồng thời 2 website trên cùng một máy chủ (Virtual Private Server) bằng kỹ thuật Virtual Host:
* **Website 1 (WordPress):** Sử dụng Source code `source_wq` và cơ sở dữ liệu `linhlt_wp_lodoz.sql`.
* **Website 2 (Laravel):** Sử dụng Source code `laravel_source` và cơ sở dữ liệu `linhlt_db.sql`.
* **Tên miền (Domains):** Sử dụng lại 2 domain đã được cấu hình trước đó.

### 1.3 Cấu hình Bảo mật & SSL
* **SSL/TLS:** Tái sử dụng các chứng chỉ SSL (Certbot/Let's Encrypt) đã tạo để đảm bảo kết nối an toàn.
* **Cấu hình luồng dữ liệu (Proxy Flow):** Đảm bảo tính nhất quán của giao thức theo yêu cầu khắt khe từ Leader:
    * `HTTP (Nginx)` ➡️ `HTTP (Apache)`
    * `HTTPS (Nginx)` ➡️ `HTTPS (Apache)`
* **Default Vhost:** Xây dựng một Virtual Host mặc định để chặn mọi truy cập trái phép qua địa chỉ IP trực tiếp hoặc các domain không xác định trỏ về VPS (Catch-all).

---

## 2. Danh mục tài liệu cung cấp

| Thành phần | Chi tiết file / Thông tin |
| :--- | :--- |
| **Laravel Data** | `linhlt_db.sql` |
| **Laravel Source** | `laravel_source` |
| **WordPress Data** | `linhlt_wp_lodoz.sql` |
| **WordPress Source** | `source_wq` |

---

## 3. Ghi chú và Lưu ý:

* **Chủ động Debug:** Quá trình triển khai thực tế sẽ phát sinh lỗi (Permission, Database Connection, Proxy Timeout...). Yêu cầu người thực hiện chủ động kiểm tra `error_log` của Nginx/Apache và bật chế độ Debug trên mã nguồn để xử lý triệt để.
* **Câu hỏi kiểm tra:** Sau khi hoàn thành, cần trả lời rõ: *Trong mô hình hiện tại, Apache đóng vai trò gì?*
* **Kiến trúc:** Đây là bước chuyển đổi quan trọng từ mô hình LEMP cơ bản sang kiến trúc chịu tải cao (High Availability), yêu cầu sự chính xác trong từng file cấu hình.

---