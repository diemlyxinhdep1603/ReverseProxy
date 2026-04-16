
```markdown
# GIAI ĐOẠN 5: CẤU HÌNH DEFAULT VHOST - BẢO MẬT TRUY CẬP IP

Yêu cầu: *"Bất kỳ domain nào khác trỏ về IP hoặc truy cập qua IP phải cần qua 1 default vhost"*. Đây là kỹ thuật thiết lập "phòng bảo vệ" để ngăn chặn các truy cập không xác định.

---

## 1. Tại sao cần Default Vhost?

* **Tình huống:** Nếu không có Default Vhost, khi ai đó gõ trực tiếp IP VPS (`221.132.21.142`) hoặc trỏ một tên miền lạ (ví dụ: `web-rac.com`) về IP của bạn, Nginx sẽ mặc định hiển thị website đầu tiên mà nó tìm thấy trong cấu hình.
* **Hậu quả:** * Gây rò rỉ dữ liệu và cấu hình hệ thống.
    * Ảnh hưởng xấu đến SEO (Duplicate Content) và uy tín của công ty nếu bị gắn với tên miền "rác".
* **Giải pháp:** Sử dụng **Catch-all Vhost** để bắt mọi truy cập sai tên miền và trả về lỗi **403 Forbidden**.

---

## 2. Các bước triển khai

### Bước 1: Khởi tạo cấu hình mặc định
Chúng ta sẽ sử dụng file `default` của Nginx để làm nơi tiếp nhận các yêu cầu không hợp lệ.

```bash
sudo nano /etc/nginx/sites-available/default
```

### Bước 2: Thiết lập luật chặn (Rule 403)
Bạn xóa sạch nội dung cũ trong file và thay bằng đoạn cấu hình chuẩn dưới đây:

```nginx
# --- KHỐI 1: CHẶN TRUY CẬP HTTP (CỔNG 80) ---
server {
    listen 80 default_server;
    server_name _;
    
    # Từ chối truy cập ngay lập tức
    return 403; 
}

# --- KHỐI 2: CHẶN TRUY CẬP HTTPS (CỔNG 443) ---
server {
    listen 443 ssl default_server;
    server_name _;
    
    # Nginx cần chứng chỉ để thiết lập kết nối SSL ban đầu trước khi từ chối.
    # Ta sử dụng tạm chứng chỉ đã có của website WordPress.
    ssl_certificate /etc/letsencrypt/live/wp.diemly.vietnix.tech/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/wp.diemly.vietnix.tech/privkey.pem;

    return 403;
}
```

### Bước 3: Kiểm tra và Áp dụng
Sau khi lưu file (`Ctrl + O`, `Enter`, `Ctrl + X`), thực hiện kiểm tra cú pháp và tải lại cấu hình:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

## 3. Giải thích từ khóa kỹ thuật (Dành cho Leader)

* **`default_server`**: Đây là từ khóa quan trọng nhất. Nó ra lệnh cho Nginx rằng: "Nếu yêu cầu của khách không khớp với bất kỳ `server_name` nào trong các website khác, hãy tống họ vào đây".
* **`server_name _`**: Dấu gạch dưới đại diện cho một tên miền không xác định hoặc truy cập trực tiếp bằng địa chỉ IP.
* **`return 403`**: Mã trạng thái HTTP Forbidden, thông báo cho người truy cập rằng họ không có quyền xem nội dung tại địa chỉ này.

---

## 4. Kiểm chứng thành quả

Ly có thể tự mình kiểm tra độ bảo mật của server bằng cách:
1. Mở trình duyệt, gõ thẳng địa chỉ IP: `http://221.132.21.142`.
2. **Kết quả chuẩn:** Màn hình phải hiển thị dòng chữ trắng trên nền đen: **"403 Forbidden"**.



Điều này chứng tỏ "tấm khiên" bảo vệ của bạn đã hoạt động hoàn hảo, giúp giấu kín các website thực tế phía sau.
```

