# HƯỚNG DẪN TỪNG BƯỚC TRIỂN KHAI STATIC HTML VỚI NGINX (DEVOPS HACKATHON)

> **Thông tin cấu hình:**
> - **Họ và tên:** Lê Phương Linh
> - **Lớp:** HNK24CNTT1
> - **Username Linux:** `lplinh-HNK24CNTT1`
> - **IP VPS:** `221.121.4.67`
> - **Mật khẩu khởi tạo mẫu:** `123456789`

---

# GIAI ĐOẠN 0: CHUẨN BỊ & KẾT NỐI BAN ĐẦU VÀO VPS

### Trường hợp 1: VPS vừa khởi tạo mới
Mở PowerShell/Terminal trên máy cá nhân:
```powershell
ssh root@221.121.4.67
```
- Nhập mật khẩu tài khoản `root`.

### Trường hợp 2: VPS vừa Rebuild / Cài lại OS (Reset)
1. Xóa cache SSH Host Key cũ trên máy cá nhân:
   ```powershell
   ssh-keygen -R 221.121.4.67
   ```
2. Đăng nhập lại vào VPS:
   ```powershell
   ssh root@221.121.4.67
   ```
   *Nhập `yes` khi được hỏi xác nhận fingerprint, sau đó nhập mật khẩu root.*

---

# PHẦN 1: QUẢN TRỊ LINUX & CÀI ĐẶT NGINX

> 💡 **LỰA CHỌN THEO ĐỀ BÀI CỦA BẠN:**
> - **TRƯỜNG HỢP A (Đề thi yêu cầu tạo User sinh viên):** Thực hiện mục [1.1](#11-trường-hợp-a-tạo-tài-khoản-linux-và-đăng-nhập-lại-theo-yêu-cầu-đề) rồi mới chuyển sang cài đặt.
> - **TRƯỜNG HỢP B (Đề bài thông thường KHÔNG yêu cầu tạo User):** Bỏ qua mục 1.1, giữ nguyên tài khoản `root` và nhảy thẳng xuống [1.2. Cài đặt Nginx, Node.js và UFW](#12-cài-đặt-nginx-nodejs-và-ufw).

---

## 1.1. TRƯỜNG HỢP A: Tạo tài khoản Linux và đăng nhập lại (Nếu đề yêu cầu)

### Bước 1: Tạo user mới (Thực hiện trên quyền `root`)
```bash
adduser --shell /bin/bash lplinh-HNK24CNTT1
```
- Nhập mật khẩu (`123456789`) 2 lần.
- Nhấn **Enter** liên tục để bỏ qua các câu hỏi bổ sung.
- Nhập `Y` rồi nhấn **Enter** để xác nhận.

### Bước 2: Thêm user vào group `sudo`
```bash
usermod -aG sudo lplinh-HNK24CNTT1
```

### Bước 3: Thoát `root` và đăng nhập lại bằng User mới
```bash
exit
```
Đăng nhập lại từ máy cá nhân:
```powershell
ssh lplinh-HNK24CNTT1@221.121.4.67
```
*(Mật khẩu: `123456789`)*

---

## 1.2. Cài đặt Nginx, Node.js và UFW (Áp dụng cho cả 2 trường hợp)
*(Nếu ở tài khoản `root`, bạn có thể gõ hoặc không gõ `sudo`; nếu ở user sinh viên thì luôn thêm `sudo`)*

### Bước 1: Cập nhật hệ thống
```bash
sudo apt update && sudo apt upgrade -y
```

### Bước 2: Cài đặt Nginx, UFW, Curl, Unzip
```bash
sudo apt install -y nginx ufw curl unzip
```

### Bước 3: Cài đặt Node.js & npm (để chạy các file kiểm tra tự động của đề)
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

### Bước 4: Kiểm tra phiên bản phần mềm
```bash
nginx -v
node --version
npm --version
ufw --version
```

---

## 1.4. Cấu hình Firewall UFW ban đầu

```bash
# Cấm toàn bộ kết nối đến mặc định
sudo ufw default deny incoming

# Cho phép kết nối đi ra
sudo ufw default allow outgoing

# Mở cổng SSH 22
sudo ufw allow 22/tcp

# Bật UFW (gõ 'y' khi được hỏi)
sudo ufw enable

# Kiểm tra trạng thái
sudo ufw status numbered
```

---

# PHẦN 2: TRIỂN KHAI WEBSITE HTML VỚI NGINX

## 2.1. Đưa mã nguồn lên VPS & Chạy kiểm tra môi trường đầu giờ

### Bước 1: Upload thư mục bài làm `devops-hackathon` lên VPS qua Bitvise SFTP
- Mở Bitvise SSH Client, đăng nhập bằng:
  - **Username:** `lplinh-HNK24CNTT1` (hoặc `root` nếu không tạo user)
  - **Port:** `22` | **Host:** `221.121.4.67`
- Mở cửa sổ **SFTP**, kéo thả toàn bộ thư mục `devops-hackathon` vào thư mục Home (`/home/lplinh-HNK24CNTT1/` hoặc `/root/`).

### Bước 2: Chạy kiểm tra môi trường ban đầu
```bash
cd ~/devops-hackathon
node environment-check.js
```

---

## 2.2. KỊCH BẢN 1: Triển khai 1 Website tĩnh đơn lẻ (Single Site)

*Ví dụ: Yêu cầu phục vụ website tại `/var/www/task-app` trên cổng `8090`.*

### Bước 1: Tạo thư mục chứa web và file `index.html`
```bash
sudo mkdir -p /var/www/task-app
sudo nano /var/www/task-app/index.html
```

Nội dung file `index.html`:
```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PTIT DevOps Task Management</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; background-color: #f4f6f9; }
        .card { background: white; padding: 25px; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        h1 { color: #2c3e50; }
        p { font-size: 16px; color: #555; }
    </style>
</head>
<body>
    <div class="card">
        <h1>PTIT DevOps Task Management</h1>
        <p><strong>Họ và tên:</strong> Lê Phương Linh</p>
        <p><strong>Mã sinh viên:</strong> HNK24CNTT1</p>
        <p><strong>Lớp:</strong> HNK24CNTT1</p>
    </div>
</body>
</html>
```

### Bước 2: Phân quyền thư mục Web cho `www-data` (Rất quan trọng)
```bash
sudo chown -R www-data:www-data /var/www/task-app
sudo chmod -R 755 /var/www/task-app
```

### Bước 3: Tạo file cấu hình Nginx
```bash
sudo nano /etc/nginx/sites-available/lplinh-HNK24CNTT1-task.conf
```

Nội dung cấu hình:
```nginx
server {
    listen 8090;
    server_name _;

    root /var/www/task-app;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

## 2.3. KỊCH BẢN 2: Triển khai Nhiều Website / Nhiều Trang (Multi-Site)

Nếu đề bài yêu cầu chạy nhiều trang web hoặc nhiều dịch vụ HTML khác nhau, có 2 phương án:

---

### 🔹 PHƯƠNG ÁN A: Chạy nhiều trang trên các CỔNG KHÁC NHAU (Multi-Port)
*Ví dụ: Trang chính ở cổng `8090` và Trang Dashboard/Admin ở cổng `8091`.*

#### Bước 1: Tạo thư mục và nội dung cho trang thứ 2 (`dashboard-app`)
```bash
sudo mkdir -p /var/www/dashboard-app
sudo nano /var/www/dashboard-app/index.html
```

Nội dung file `index.html` của trang 2:
```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Dashboard - Lê Phương Linh</title>
</head>
<body>
    <h1>Trang Quản Trị Hệ Thống - Dashboard</h1>
    <p>Sinh viên: Lê Phương Linh - Lớp: HNK24CNTT1</p>
    <p>Trạng thái: Đang hoạt động trên cổng 8091</p>
</body>
</html>
```

#### Bước 2: Phân quyền thư mục trang 2
```bash
sudo chown -R www-data:www-data /var/www/dashboard-app
sudo chmod -R 755 /var/www/dashboard-app
```

#### Bước 3: Tạo file cấu hình Nginx cho trang thứ 2
```bash
sudo nano /etc/nginx/sites-available/lplinh-HNK24CNTT1-dashboard.conf
```

Nội dung cấu hình:
```nginx
server {
    listen 8091;
    server_name _;

    root /var/www/dashboard-app;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

### 🔹 PHƯƠNG ÁN B: Chạy nhiều trang theo ĐƯỜNG DẪN CON (Subpath / Subdirectory)
*Ví dụ: Cùng chạy trên cổng `8090`:*
- `http://221.121.4.67:8090/` -> Mở trang chính tại `/var/www/task-app/`
- `http://221.121.4.67:8090/dashboard/` -> Mở trang dashboard tại `/var/www/dashboard-app/`

#### Cấu hình Nginx gộp chung trong 1 file `.conf`:
```bash
sudo nano /etc/nginx/sites-available/lplinh-HNK24CNTT1-task.conf
```

Điền nội dung cấu hình sử dụng `alias`:
```nginx
server {
    listen 8090;
    server_name _;

    # 1. Trang chính tại root /
    location / {
        root /var/www/task-app;
        index index.html index.htm;
        try_files $uri $uri/ =404;
    }

    # 2. Trang phụ tại đường dẫn /dashboard/ (Bắt buộc dùng alias và có dấu / ở cuối)
    location /dashboard/ {
        alias /var/www/dashboard-app/;
        index index.html index.htm;
        try_files $uri $uri/ =404;
    }
}
```

---

# PHẦN 3: KÍCH HOẠT, KIỂM TRA & MỞ CỔNG FIREWALL

## 3.1. Kích hoạt Virtual Host & Kiểm tra cú pháp Nginx

### Bước 1: Tạo liên kết kích hoạt (Symlink) sang `sites-enabled`
```bash
# Kích hoạt file cấu hình trang chính
sudo ln -s /etc/nginx/sites-available/lplinh-HNK24CNTT1-task.conf /etc/nginx/sites-enabled/

# (Nếu có file cấu hình trang thứ 2 theo cổng riêng)
sudo ln -s /etc/nginx/sites-available/lplinh-HNK24CNTT1-dashboard.conf /etc/nginx/sites-enabled/
```

### Bước 2: Kiểm tra tính đúng đắn của toàn bộ cấu hình Nginx
```bash
sudo nginx -t
```
*Kết quả đạt được chuẩn xác:*
```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

---

## 3.2. Mở cổng Firewall tương ứng & Nạp lại Nginx

```bash
# Mở cổng 8090 (và 8091 nếu dùng phương án Multi-Port)
sudo ufw allow 8090/tcp
sudo ufw allow 8091/tcp

# Nạp lại Nginx
sudo systemctl reload nginx

# Kiểm tra lại bảng Firewall
sudo ufw status numbered
```

---

## 3.3. Kiểm tra thực tế

1. **Kiểm tra từ Terminal bằng `curl`:**
   ```bash
   curl -I http://127.0.0.1:8090/
   ```
   *Kết quả:* Trả về mã `HTTP/1.1 200 OK`.

2. **Kiểm tra trên Trình duyệt máy cá nhân:**
   - Truy cập: `http://221.121.4.67:8090/` -> Hiển thị trang thông tin sinh viên Lê Phương Linh.
   - Nếu có Dashboard: `http://221.121.4.67:8091/` (hoặc `http://221.121.4.67:8090/dashboard/`).

---

# PHẦN 4: ĐÓNG GÓI MINH CHỨNG & CHẠY SYSTEM INSPECTOR (10 ĐIỂM)

## 4.1. Chuẩn bị thư mục `submission/`
```bash
cd ~/devops-hackathon
mkdir -p submission
```

## 4.2. Copy các file cấu hình Nginx vào `submission/`
```bash
# Copy các file cấu hình đã tạo vào thư mục nộp bài
cp /etc/nginx/sites-available/lplinh-HNK24CNTT1-*.conf ~/devops-hackathon/submission/
```

## 4.3. Chạy công cụ kiểm tra tự động cuối bài (System Inspector)
```bash
cd ~/devops-hackathon
node system-inspector.js
```
*Kết quả đạt được:* Hệ thống thông báo tạo thành công file: `submission/report.enc`.

## 4.4. Lưu lại toàn bộ lịch sử dòng lệnh (History)
```bash
history > ~/devops-hackathon/submission/history.log
```

---

# PHẦN 5: TẢI BÀI NỘP QUA SFTP & CHECKLIST KIỂM TRA

## 5.1. Tải thư mục bài làm về máy cá nhân
1. Mở cửa sổ **Bitvise SFTP**.
2. Ở khung bên phải, vào thư mục Home (`/home/lplinh-HNK24CNTT1/` hoặc `/root/`).
3. Tải toàn bộ thư mục `devops-hackathon` về máy tính.
4. Kiểm tra cấu trúc thư mục nộp bài đầy đủ:
```text
devops-hackathon/
├── templates/
├── submission/
│   ├── history.log
│   ├── lplinh-HNK24CNTT1-task.conf
│   └── report.enc
├── environment-check.js
└── system-inspector.js
```

---

## 5.2. Bảng tổng hợp checklist kiểm tra trước khi nộp bài

| STT | Hạng mục kiểm tra | Lệnh thực hiện | Kết quả mong muốn |
| :--- | :--- | :--- | :--- |
| **1** | Tài khoản thực hiện | `whoami` | `lplinh-HNK24CNTT1` (Không phải root) |
| **2** | Quyền thư mục Web | `ls -ld /var/www/task-app` | `drwxr-xr-x www-data www-data` |
| **3** | Cú pháp Nginx | `sudo nginx -t` | `syntax is ok` & `test is successful` |
| **4** | Tình trạng Nginx | `sudo systemctl is-active nginx` | `active` |
| **5** | UFW Firewall | `sudo ufw status numbered` | Allow 22, Allow 8090 (và 8091) |
| **6** | Web Response | `curl -I http://127.0.0.1:8090` | `HTTP/1.1 200 OK` |
| **7** | Minh chứng nộp bài | `ls -la ~/devops-hackathon/submission/` | Đủ file `history.log`, `report.enc`, `.conf` |
