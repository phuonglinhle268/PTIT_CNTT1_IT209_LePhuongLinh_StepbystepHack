# CẨM NANG TOÀN DIỆN: TRIỂN KHAI WEBSITE HTML/CSS VỚI NGINX TRÊN LINUX VPS

> **Thông tin cấu hình thực hành:**
> - **Họ và tên:** Lê Phương Linh
> - **Lớp:** HNK24CNTT1
> - **Tài khoản Linux User:** `lplinh-HNK24CNTT1` *(nếu đề yêu cầu tạo)*
> - **Địa chỉ IP VPS:** `221.121.4.67`
> - **Cổng kết nối SSH:** `22`
> - **Mật khẩu khởi tạo mẫu:** `123456789`

---

# HƯỚNG DẪN CHI TIẾT SỬ DỤNG PHẦN MỀM BITVISE SSH CLIENT (CHO NGƯỜI MỚI BẮT ĐẦU)

Bitvise SSH Client là công cụ đồ họa giúp bạn vừa gõ lệnh Linux (Terminal), vừa kéo thả file giữa máy tính và VPS (SFTP) một cách trực quan.

### 1. Cách đăng nhập vào VPS qua Bitvise
1. Khởi động phần mềm **Bitvise SSH Client** trên máy tính Windows.
2. Tại tab **Login** (ở giữa màn hình), bạn điền chính xác:
   - **Host:** `221.121.4.67`
   - **Port:** `22`
   - **Username:** `root` *(hoặc `lplinh-HNK24CNTT1` nếu đã tạo user)*
   - **Initial method:** chọn `password`
   - **Password:** nhập mật khẩu (ví dụ `123456789` hoặc mật khẩu root của VPS)
   - Tích chọn ô vuông: `Store encrypted password in profile` *(để lần sau không cần gõ lại)*.
3. Nhấp vào nút **`Log in`** ở góc dưới cùng bên trái.
4. **Xử lý thông báo khóa Host Key:**
   - Nếu xuất hiện bảng popup cảnh báo *Host Key Verification*, bạn nhấp nút **`Accept and Save`**.
5. **Dấu hiệu đăng nhập thành công:**
   - Dòng chữ dưới đáy phần mềm chuyển sang trạng thái: `Authentication completed. Session started.`

---

### 2. Cách mở Terminal và SFTP trên Bitvise
Nhìn vào thanh menu màu xám ở cột bên trái của Bitvise:
- **Mở cửa sổ gõ lệnh Linux:** Nhấp chuột vào nút **`New terminal console`** (hoặc biểu tượng màn hình đen). Cửa sổ dòng lệnh màu đen sẽ hiện ra để bạn gõ các câu lệnh.
- **Mở cửa sổ truyền nhận file (SFTP):** Nhấp chuột vào nút **`New SFTP window`** (hoặc biểu tượng 2 thư mục). 

---

### 3. Hướng dẫn thao tác kéo thả File trên cửa sổ SFTP
Khi cửa sổ SFTP mở ra, bạn sẽ thấy chia làm 2 nửa màn hình:
- **Khung bên trái (Local files):** Là các ổ đĩa `C:`, `D:`, thư mục trên máy tính Windows cá nhân của bạn.
- **Khung bên phải (Remote files):** Là các thư mục nằm trên máy chủ VPS Linux.

**Thao tác Upload (Gửi file từ máy tính lên VPS):**
1. Ở khung bên trái, nhấp đúp chuột tìm đến thư mục chứa bài làm `devops-hackathon` trên máy tính của bạn.
2. Ở khung bên phải, đảm bảo đang ở thư mục Home của bạn (ví dụ `/root/` hoặc `/home/lplinh-HNK24CNTT1/`).
3. Giữ chuột trái vào thư mục `devops-hackathon` ở bên trái, **kéo và thả sang khung bên phải**. Quá trình tải lên sẽ chạy trong vài giây.

**Thao tác Download (Tải bài từ VPS về máy tính để nộp):**
1. Ở khung bên phải (Remote), tìm thư mục `devops-hackathon`.
2. Giữ chuột trái vào thư mục `devops-hackathon`, **kéo và thả sang khung bên trái** (về máy tính cá nhân).

---

# GIAI ĐOẠN 0: XỬ LÝ KHI VỪA RESET / CÀI LẠI OS MÁY ẢO

Nếu bạn vừa bấm **Reset / Cài lại OS** trên trang quản lý VPS, mã bảo mật SSH cũ trên máy tính của bạn sẽ bị lệch, khi SSH sẽ báo lỗi `Host key verification failed`.

### Cách khắc phục:
1. Mở **PowerShell** trên máy tính cá nhân (hoặc trong Bitvise bấm Logout rồi Login lại).
2. Chạy lệnh xóa cache key cũ:
   ```powershell
   ssh-keygen -R 221.121.4.67
   ```
   *Kết quả thành công:* Hệ thống hiển thị:
   `# Host 221.121.4.67 found: line ... updated / ... (key removed)`
3. Kết nối lại bình thường bằng Bitvise hoặc lệnh:
   ```powershell
   ssh root@221.121.4.67
   ```

---

# PHẦN 1: QUẢN TRỊ LINUX & CÀI ĐẶT NGINX

> 💡 **LỰA CHỌN THEO ĐỀ BÀI CỦA BẠN:**
> - **TRƯỜNG HỢP A (Đề thi yêu cầu tạo User sinh viên `[tên]-[lớp]`):** Bắt buộc làm mục [1.1](#11-trường-hợp-a-tạo-user-sinh-viên-và-đăng-nhập-lại-nếu-đề-yêu-cầu).
> - **TRƯỜNG HỢP B (Đề bài KHÔNG yêu cầu tạo User):** Bỏ qua mục 1.1, giữ nguyên tài khoản `root` và nhảy thẳng xuống [1.2. Cài đặt môi trường Nginx](#12-cài-đặt-nginx-nodejs-và-ufw-áp-dụng-cho-cả-2-trường-hợp).

---

## 1.1. TRƯỜNG HỢP A: Tạo User sinh viên và đăng nhập lại (Nếu đề yêu cầu)

### Bước 1: Tạo user mới trên Terminal (đang ở quyền `root`)
Gõ lệnh sau và nhấn Enter:
```bash
adduser --shell /bin/bash lplinh-HNK24CNTT1
```
*Màn hình sẽ hiển thị các câu hỏi từng bước:*
- `New password:` -> Gõ `123456789` rồi nhấn **Enter** *(Lưu ý: Linux sẽ ẩn mật khẩu, không hiện dấu sao, bạn cứ gõ bình thường)*.
- `Retype new password:` -> Gõ lại `123456789` rồi nhấn **Enter**.
- `Full Name []:` -> Nhấn **Enter** để bỏ qua.
- `Room Number []:` -> Nhấn **Enter** để bỏ qua.
- `Work Phone []:` -> Nhấn **Enter** để bỏ qua.
- `Home Phone []:` -> Nhấn **Enter** để bỏ qua.
- `Other []:` -> Nhấn **Enter** để bỏ qua.
- `Is the information correct? [Y/n]` -> Gõ `Y` rồi nhấn **Enter**.

### Bước 2: Cấp quyền quản trị `sudo` cho user mới
```bash
usermod -aG sudo lplinh-HNK24CNTT1
```
*Kết quả:* Lệnh chạy ngầm và trả về dấu nhắc lệnh mới, không báo lỗi.

### Bước 3: Kiểm tra phân quyền user
```bash
id lplinh-HNK24CNTT1
```
*Kết quả xuất hiện trên màn hình:*
`uid=1001(lplinh-HNK24CNTT1) gid=1001(lplinh-HNK24CNTT1) groups=1001(lplinh-HNK24CNTT1),27(sudo)`

### Bước 4: Thoát Root và Đăng nhập lại bằng User mới
Gõ lệnh thoát:
```bash
exit
```
- Mở Bitvise: Đổi **Username** thành `lplinh-HNK24CNTT1`, nhập mật khẩu `123456789` rồi bấm **Log in** và mở lại **New terminal console**.
- Hoặc từ PowerShell gõ:
```powershell
ssh lplinh-HNK24CNTT1@221.121.4.67
```

---

## 1.2. Cài đặt Nginx, Node.js và UFW (Áp dụng cho cả 2 trường hợp)

*(Nếu bạn đang ở quyền `root`, các lệnh có thể bỏ chữ `sudo`; nếu ở user sinh viên thì giữ nguyên `sudo`)*

### Bước 1: Cập nhật danh sách phần mềm hệ thống
```bash
sudo apt update && sudo apt upgrade -y
```
*Kết quả:* Hệ thống tải danh sách gói, hiển thị `Reading package lists... Done` và `All packages are up to date`.

### Bước 2: Cài đặt Web Server Nginx và các tiện ích cần thiết
```bash
sudo apt install -y nginx ufw curl unzip
```
*Kết quả:* Nginx và UFW được cài đặt, kết thúc bằng dòng `Setting up nginx ...`

### Bước 3: Cài đặt Node.js và npm (để chạy công cụ kiểm tra tự động)
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

### Bước 4: Kiểm tra phiên bản các công cụ đã cài đặt
```bash
nginx -v
node --version
npm --version
ufw --version
```
*Kết quả hiển thị tương tự:*
```text
nginx version: nginx/1.18.0 (Ubuntu)
v20.18.0
10.8.2
ufw 0.36.2
```

---

## 1.3. Cấu hình Tường lửa (Firewall UFW)

### Bước 1: Thiết lập chính sách bảo mật
```bash
# 1. Chặn toàn bộ kết nối đến mặc định
sudo ufw default deny incoming

# 2. Cho phép kết nối đi ra ngoài
sudo ufw default allow outgoing

# 3. Mở cổng SSH 22 (Bắt buộc để không bị mất kết nối)
sudo ufw allow 22/tcp
```
*Kết quả mỗi lệnh:* Hệ thống báo `Rules updated`.

### Bước 2: Kích hoạt Firewall
```bash
sudo ufw enable
```
*Terminal hỏi:* `Command may disrupt existing ssh connections. Proceed with operation (y|n)?`
-> Bạn gõ `y` rồi nhấn **Enter**.
*Kết quả:* `Firewall is active and enabled on system startup`.

### Bước 3: Kiểm tra trạng thái Firewall
```bash
sudo ufw status numbered
```
*Kết quả bảng tường lửa chuẩn:*
```text
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
```

---

# PHẦN 2: TRIỂN KHAI WEBSITE HTML VỚI NGINX

## 2.1. Upload mã nguồn & Chạy kiểm tra ban đầu

1. Dùng **Bitvise SFTP** kéo thả toàn bộ thư mục `devops-hackathon` vào thư mục Home trên VPS.
2. Trên Terminal, di chuyển vào thư mục dự án và chạy file kiểm tra môi trường:
```bash
cd ~/devops-hackathon
node environment-check.js
```
*Kết quả:* Xuất hiện danh sách các mục kiểm tra tích xanh `[PASS]` cho Node, Nginx, UFW.

---

## 2.2. KỊCH BẢN 1: Triển khai 1 Trang Web tĩnh Đơn Lẻ (Single Site)

*Giả sử đề yêu cầu: Phục vụ website tại thư mục `/var/www/task-app` trên cổng `8090`.*

### Bước 1: Tạo thư mục Web và tạo file `index.html`
```bash
sudo mkdir -p /var/www/task-app
sudo nano /var/www/task-app/index.html
```
> **Mẹo dùng `nano` trên Terminal:**
> - Sau khi gõ lệnh trên, màn hình soạn thảo mở ra.
> - Copy toàn bộ đoạn HTML bên dưới -> Click chuột phải vào màn hình terminal để Dán (Paste).
> - Nhấn tổ hợp phím **`Ctrl + O`** -> nhấn **`Enter`** (để Lưu file).
> - Nhấn tổ hợp phím **`Ctrl + X`** (để Thoát ra ngoài).

**Nội dung file `index.html`:**
```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PTIT DevOps Task Management</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; background-color: #f4f6f9; }
        .card { background: white; padding: 25px; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); max-width: 600px; margin: auto; }
        h1 { color: #2c3e50; border-bottom: 2px solid #3498db; padding-bottom: 10px; }
        p { font-size: 16px; color: #333; line-height: 1.6; }
    </style>
</head>
<body>
    <div class="card">
        <h1>PTIT DevOps Task Management</h1>
        <p><strong>Họ và tên:</strong> Lê Phương Linh</p>
        <p><strong>Mã sinh viên:</strong> HNK24CNTT1</p>
        <p><strong>Lớp:</strong> HNK24CNTT1</p>
        <p><strong>Trạng thái:</strong> Website HTML chạy hoàn hảo trên Nginx</p>
    </div>
</body>
</html>
```

### Bước 2: Phân quyền thư mục Web cho user `www-data` (Cực kỳ quan trọng)
```bash
sudo chown -R www-data:www-data /var/www/task-app
sudo chmod -R 755 /var/www/task-app
```
*Kết quả:* Không có lỗi xuất hiện. Kiểm tra bằng lệnh: `ls -ld /var/www/task-app` sẽ thấy chủ sở hữu là `www-data www-data`.

### Bước 3: Tạo file cấu hình Virtual Host Nginx
```bash
sudo nano /etc/nginx/sites-available/lplinh-HNK24CNTT1-task.conf
```
*(Nếu đề không yêu cầu đặt tên theo user, bạn có thể đặt `task-app.conf`)*

Dán nội dung cấu hình sau:
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
*(Bấm `Ctrl + O` -> `Enter` để lưu, `Ctrl + X` để thoát)*

---

## 2.3. KỊCH BẢN 2: Triển khai Nhiều Website / Nhiều Trang (Multi-Site)

Nếu đề bài yêu cầu chạy từ 2 website trở lên, bạn có 2 phương án tùy theo yêu cầu cụ thể của đề:

---

### 🔹 PHƯƠNG ÁN A: Chạy nhiều trang trên các CỔNG KHÁC NHAU (Multi-Port)
*Ví dụ: Trang 1 ở cổng `8090`, Trang 2 (Dashboard) ở cổng `8091`.*

#### Bước 1: Tạo thư mục và nội dung cho trang thứ 2
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
    <title>Trang Dashboard - Lê Phương Linh</title>
</head>
<body>
    <h1>Trang Quản Trị Hệ Thống (Dashboard)</h1>
    <p>Sinh viên thực hiện: Lê Phương Linh - Lớp: HNK24CNTT1</p>
    <p>Trang này đang phục vụ độc lập tại cổng <strong>8091</strong>.</p>
</body>
</html>
```

#### Bước 2: Phân quyền cho trang thứ 2
```bash
sudo chown -R www-data:www-data /var/www/dashboard-app
sudo chmod -R 755 /var/www/dashboard-app
```

#### Bước 3: Tạo file cấu hình Nginx riêng cho trang thứ 2
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

### 🔹 PHƯƠNG ÁN B: Chạy nhiều trang theo ĐƯỜNG DẪN CON (Subpath bằng `alias`)
*Ví dụ: Cùng chạy trên cổng `8090`:*
- `http://221.121.4.67:8090/` -> Mở trang chính tại `/var/www/task-app`
- `http://221.121.4.67:8090/dashboard/` -> Mở trang phụ tại `/var/www/dashboard-app`

Chỉnh sửa file cấu hình chính:
```bash
sudo nano /etc/nginx/sites-available/lplinh-HNK24CNTT1-task.conf
```
Cấu hình như sau (sử dụng directive `alias`):
```nginx
server {
    listen 8090;
    server_name _;

    # 1. Trang chính ở đường dẫn gốc /
    location / {
        root /var/www/task-app;
        index index.html index.htm;
        try_files $uri $uri/ =404;
    }

    # 2. Trang phụ ở đường dẫn /dashboard/ (Lưu ý dấu / ở cuối)
    location /dashboard/ {
        alias /var/www/dashboard-app/;
        index index.html index.htm;
        try_files $uri $uri/ =404;
    }
}
```

---

# PHẦN 3: KÍCH HOẠT, KIỂM TRA & MỞ CỔNG FIREWALL

## 3.1. Kích hoạt Virtual Host Nginx

Tạo liên kết tượng trưng (Symlink) từ `sites-available` sang `sites-enabled`:
```bash
# Kích hoạt site chính
sudo ln -s /etc/nginx/sites-available/lplinh-HNK24CNTT1-task.conf /etc/nginx/sites-enabled/

# Kích hoạt site thứ 2 (nếu bạn làm theo Phương án A - Cổng 8091)
sudo ln -s /etc/nginx/sites-available/lplinh-HNK24CNTT1-dashboard.conf /etc/nginx/sites-enabled/
```

## 3.2. Kiểm tra cú pháp cấu hình Nginx (Bắt buộc)
```bash
sudo nginx -t
```
*Kết quả đạt chuẩn tuyệt đối:*
```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
> ⚠️ *Nếu báo lỗi dòng nào, bạn mở lại file `.conf` kiểm tra xem có quên dấu chấm phẩy `;` hoặc sai đường dẫn không.*

---

## 3.3. Mở cổng Firewall & Nạp lại cấu hình Nginx

### Bước 1: Mở cổng web trên UFW
```bash
# Mở cổng 8090
sudo ufw allow 8090/tcp

# Mở cổng 8091 (nếu có dùng thêm cổng 8091)
sudo ufw allow 8091/tcp
```
*Kết quả:* `Rules updated`.

### Bước 2: Nạp lại cấu hình Nginx
```bash
sudo systemctl reload nginx
```
*Kết quả:* Lệnh chạy xong ngay lập tức và không có lỗi.

### Bước 3: Kiểm tra trạng thái dịch vụ Nginx
```bash
sudo systemctl status nginx
```
*Kết quả:* Nhìn thấy chữ màu xanh lá cây: **`Active: active (running)`**. Nhấn **`q`** để thoát.

---

## 3.4. Kiểm tra hoạt động thực tế

1. **Kiểm tra từ dòng lệnh bằng `curl`:**
   ```bash
   curl -I http://127.0.0.1:8090/
   ```
   *Kết quả phản hồi thành công:*
   ```text
   HTTP/1.1 200 OK
   Server: nginx/...
   Content-Type: text/html
   ```

2. **Kiểm tra trên Trình duyệt máy tính cá nhân (Chrome/Edge):**
   - Mở trình duyệt và gõ địa chỉ: `http://221.121.4.67:8090/`
   -> Hiển thị trang web HTML có tên *Lê Phương Linh - HNK24CNTT1*.
   - Nếu có trang phụ: truy cập `http://221.121.4.67:8091/` hoặc `http://221.121.4.67:8090/dashboard/`.

---

# PHẦN 4: ĐÓNG GÓI MINH CHỨNG & CHẠY SYSTEM INSPECTOR (10 ĐIỂM)

## 4.1. Tạo thư mục `submission/`
```bash
cd ~/devops-hackathon
mkdir -p submission
```

## 4.2. Copy các file cấu hình Nginx vào `submission/`
```bash
# Copy toàn bộ file cấu hình site vừa tạo vào thư mục nộp bài
cp /etc/nginx/sites-available/lplinh-HNK24CNTT1-*.conf ~/devops-hackathon/submission/
```

## 4.3. Chạy công cụ kiểm tra và tạo file báo cáo tự động (System Inspector)
```bash
cd ~/devops-hackathon
node system-inspector.js
```
*Kết quả đạt được:*
Hệ thống chạy quét toàn bộ dịch vụ và hiển thị thông báo tạo thành công file: `submission/report.enc`.

## 4.4. Lưu lại toàn bộ lịch sử dòng lệnh (History)
```bash
history > ~/devops-hackathon/submission/history.log
```

## 4.5. Kiểm tra các file đã có trong thư mục nộp bài
```bash
ls -la ~/devops-hackathon/submission/
```
*Kết quả nhìn thấy đầy đủ:*
- `history.log`
- `report.enc`
- `lplinh-HNK24CNTT1-task.conf` (hoặc các file `.conf` bạn đã tạo)

---

# PHẦN 5: TẢI BÀI NỘP VỀ MÁY QUA BITVISE SFTP & CHECKLIST

### Các bước tải bài về máy tính:
1. Mở cửa sổ **Bitvise SFTP**.
2. Ở khung bên phải (Remote), mở thư mục Home (`/home/lplinh-HNK24CNTT1/` hoặc `/root/`).
3. Nhấp giữ chuột trái vào thư mục `devops-hackathon`, kéo thả sang khung bên trái (máy tính của bạn).
4. Kiểm tra cấu trúc thư mục đã tải về máy đảm bảo đầy đủ:
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

## BẢNG TỔNG HỢP KIỂM TRA NHANH TRƯỚC KHI NỘP BÀI

| STT | Nội dung kiểm tra | Lệnh chạy trên Terminal | Kết quả đạt chuẩn |
| :--- | :--- | :--- | :--- |
| **1** | Tài khoản thực hiện | `whoami` | `lplinh-HNK24CNTT1` (hoặc `root` nếu không tạo user) |
| **2** | Quyền thư mục Web | `ls -ld /var/www/task-app` | `drwxr-xr-x ... www-data www-data` |
| **3** | Cú pháp Nginx | `sudo nginx -t` | `syntax is ok` & `test is successful` |
| **4** | Trạng thái Nginx | `sudo systemctl is-active nginx` | `active` |
| **5** | Firewall UFW | `sudo ufw status numbered` | Cổng 22, 8090 (và 8091) đều `ALLOW IN` |
| **6** | Phản hồi Web Server | `curl -I http://127.0.0.1:8090` | Trả về mã `HTTP/1.1 200 OK` |
| **7** | Minh chứng nộp bài | `ls -la ~/devops-hackathon/submission/` | Đủ file `history.log`, `report.enc`, `.conf` |
