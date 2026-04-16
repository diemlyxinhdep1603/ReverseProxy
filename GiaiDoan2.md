


# GIAI ĐOẠN 2: CÀI ĐẶT APACHE & CẤU HÌNH ĐỔI CỔNG (PHÂN TÁCH NHIỆM VỤ)

🚨 **Lưu ý quan trọng:** Hiện tại Nginx đang chiếm giữ các cổng tiêu chuẩn là 80 và 443. Để xây dựng mô hình Reverse Proxy, chúng ta phải cấu hình Apache rút về "hậu trường" bằng cách thay đổi cổng lắng nghe trước khi kích hoạt dịch vụ.

---

## 1. Tư duy hệ thống: Quy luật "Thêm số" 

Để hiểu tại sao lại chọn các con số **8080** và **8443**, chúng ta cần dựa trên các cổng chuẩn quốc tế:
* **HTTP (Cổng 80):** Web bình thường ➡️ Đôn lên thành **8080**.
* **HTTPS (Cổng 443):** Web bảo mật có SSL ➡️ Thêm số 8 vào đầu thành **8443**.

**Vai trò trong mô hình:** * **Nginx:** Đóng vai trò là "Lễ tân" đón khách trực tiếp ở cửa ngoài (Cổng 80/443).
* **Apache:** Đóng vai trò là "Bếp trưởng" xử lý món ăn khó ở phía sau (Cổng 8080/8443).

### Tóm tắt luồng đi theo yêu cầu:
* **Luồng 1 (Không bảo mật):** Khách truy cập `http://` ➡️ Nginx (80) ➡️ Apache (8080).
* **Luồng 2 (Có bảo mật):** Khách truy cập `https://` ➡️ Nginx (443) ➡️ Apache (8443).

---

## 2. Các bước triển khai chi tiết

### Bước 1: Cài đặt Apache và Module PHP
Thực hiện cài đặt dịch vụ Apache và bộ thư viện giúp Apache hiểu được ngôn ngữ PHP 8.1.

```bash
sudo apt update && sudo apt install apache2 libapache2-mod-php8.1 -y
```

> 🚨 **CẢNH BÁO:** Sau khi cài xong, hệ thống có thể báo lỗi màu đỏ (như *Job for apache2.service failed*). Đây là hiện tượng **BÌNH THƯỜNG** vì Apache vừa cài xong sẽ mặc định chiếm cổng 80 nhưng bị Nginx chặn lại. Chúng ta sẽ fix lỗi này ở Bước 3.

### Bước 2: Kích hoạt các tính năng bổ trợ (Modules)
Để Apache xử lý được HTTPS và đọc được các file cấu hình nâng cao của Laravel/WordPress (`.htaccess`), chúng ta cần "đánh thức" các module này:

```bash
sudo a2enmod rewrite ssl headers
```

### Bước 3: Cấu hình đổi cổng Apache (Rút về hậu trường)
Đây là bước then chốt để giải quyết xung đột với Nginx. Chúng ta sẽ chỉnh sửa file `ports.conf`.

```bash
sudo nano /etc/apache2/ports.conf
```

**Nội dung chỉnh sửa chính xác:**
1. Tìm dòng `Listen 80` ➡️ Đổi thành `Listen 8080`.
2. Trong khối `<IfModule ssl_module>`, tìm dòng `Listen 443` ➡️ Đổi thành `Listen 8443`.

*Cấu hình sau khi sửa sẽ trông như sau:*
```apache
Listen 8080

<IfModule ssl_module>
    Listen 8443
</IfModule>

<IfModule mod_gnutls.c>
    Listen 443
</IfModule>
```
*(Nhấn `Ctrl + O` ➡️ `Enter` để lưu, `Ctrl + X` để thoát).*

---

## 3. Khởi động và Kiểm tra trạng thái

Sau khi đã nhường cổng chuẩn cho Nginx, chúng ta khởi động lại Apache để áp dụng cấu hình mới.

```bash
sudo systemctl restart apache2
sudo systemctl status apache2
```

### Kết quả kiểm chứng:
Hệ thống phải hiển thị dòng chữ **Active: active (running)** màu xanh lá cây. Điều này chứng tỏ Apache đã khởi động thành công trên các cổng phụ mà không còn xung đột với Nginx.



