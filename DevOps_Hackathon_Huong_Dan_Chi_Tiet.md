# HƯỚNG DẪN TỪNG BƯỚC THỰC HIỆN DEVOPS HACKATHON
> **Thông tin thí sinh:**
> - **Họ và tên:** Lê Phương Linh
> - **Lớp:** HNK24CNTT1
> - **Tên tài khoản Linux (Username):** `lplinh-HNK24CNTT1` *(hoặc `linhlp-HNK24CNTT1` theo quy ước của bạn)*
> - **Địa chỉ IP VPS:** `221.121.4.67`
> - **Mật khẩu khởi tạo mẫu (Linux/MySQL):** `Devops@2026` *(bạn có thể tự đặt mật khẩu riêng)*

---

## MỤC LỤC
1. [Giai đoạn 0: Chuẩn bị & Kết nối ban đầu vào VPS](#giai-đoạn-0-chuẩn-bị--kết-nối-ban-đầu-vào-vps)
2. [Phần 1: Quản trị Linux & Cấu hình Firewall (35 điểm)](#phần-1-quản-trị-linux--cấu-hình-firewall-35-điểm)
   - 1.1. Tạo Linux User & Phân quyền Sudo
   - 1.2. Thoát Root & Đăng nhập bằng User mới
   - 1.3. Cài đặt toàn bộ môi trường (Node, Java, Gradle, Nginx, MySQL, UFW)
   - 1.4. Cấu hình tường lửa UFW
3. [Phần 2: Triển khai Backend Java & Database MySQL (55 điểm)](#phần-2-triển-khai-backend-java--database-mysql-55-điểm)
   - 2.1. Upload mã nguồn & Chạy `environment-check.js`
   - 2.2. Cấu hình & Import MySQL Database
   - 2.3. Cấu hình & Build Backend bằng IntelliJ / Gradle
   - 2.4. Chạy thử nghiệm Backend
   - 2.5. Tạo và kích hoạt Systemd Service
4. [Phần 3: Cấu hình Nginx Reverse Proxy & Static Web (10 điểm)](#phần-3-cấu-hình-nginx-reverse-proxy--static-web-10-điểm)
   - 3.1. Tạo Static Website thông tin sinh viên
   - 3.2. Cấu hình Nginx Reverse Proxy cổng 8090
   - 3.3. Kích hoạt và mở cổng UFW 8090
5. [Phần 4: Đóng gói minh chứng & Chạy System Inspector (10 điểm)](#phần-4-đóng-gói-minh-chứng--chạy-system-inspector-10-điểm)
6. [Phần 5: Tải bài nộp qua Bitvise SFTP & Checklist kiểm tra](#phần-5-tải-bài-nộp-qua-bitvise-sftp--checklist-kiểm-tra)

---

# GIAI ĐOẠN 0: CHUẨN BỊ & KẾT NỐI BAN ĐẦU VÀO VPS

### Trường hợp 1: VPS vừa được khởi tạo mới hoàn toàn
Mở PowerShell/Terminal trên máy tính cá nhân và chạy:
```powershell
ssh root@221.121.4.67
```
- Nhập mật khẩu tài khoản `root` được cấp.

### Trường hợp 2: VPS vừa được Rebuild / Cài lại OS (Reset)
Khi Rebuild OS, SSH Fingerprint cũ trên máy tính của bạn sẽ bị lệch và gây lỗi `Host key verification failed`.
1. Chạy lệnh xóa cache key cũ trên PowerShell máy cá nhân:
   ```powershell
   ssh-keygen -R 221.121.4.67
   ```
   *Kết quả thành công:* Hệ thống báo `Updated ... (key removed)`.
2. Đăng nhập lại vào VPS:
   ```powershell
   ssh root@221.121.4.67
   ```
   *Khi có câu hỏi `Are you sure you want to continue connecting (yes/no)?`, gõ `yes` và nhấn Enter, sau đó nhập mật khẩu root.*

---

# PHẦN 1: QUẢN TRỊ LINUX & FIREWALL (35 ĐIỂM)

## 1.1. Tạo tài khoản Linux và phân quyền Sudo (Thực hiện bằng quyền `root`)

### Bước 1: Tạo user mới với shell mặc định `/bin/bash`
```bash
adduser --shell /bin/bash lplinh-HNK24CNTT1
```
- Nhập mật khẩu cho user (ví dụ: `Devops@2026`), nhập lại 2 lần.
- Các thông tin bổ sung (`Full Name`, `Room Number`...) nhấn **Enter** liên tục để bỏ qua.
- Nhập `Y` rồi nhấn **Enter** để xác nhận tạo.

### Bước 2: Thêm user vào group `sudo`
```bash
usermod -aG sudo lplinh-HNK24CNTT1
```

### Bước 3: Kiểm tra thông tin phân quyền
```bash
id lplinh-HNK24CNTT1
```
*Kết quả đạt được mong muốn:*
`uid=1001(lplinh-HNK24CNTT1) gid=1001(lplinh-HNK24CNTT1) groups=1001(lplinh-HNK24CNTT1),27(sudo)`

---

## 1.2. ⚠️ BẮT BUỘC: Thoát Root và Đăng nhập lại bằng User mới

### Bước 1: Thoát tài khoản `root`
```bash
exit
```

### Bước 2: Đăng nhập lại VPS từ PowerShell bằng User mới tạo
```powershell
ssh lplinh-HNK24CNTT1@221.121.4.67
```
*(Nhập mật khẩu `Devops@2026` của user `lplinh-HNK24CNTT1`)*

> ⛔ **Lưu ý quan trọng:** Từ thời điểm này trở đi, **toàn bộ các thao tác còn lại** tuyệt đối không được dùng tài khoản `root` để tránh bị trừ điểm. Các lệnh cần quyền quản trị chỉ sử dụng tiền tố `sudo`.

---

## 1.3. Cài đặt toàn bộ môi trường phần mềm (5 điểm)

### Bước 1: Cập nhật hệ thống
```bash
sudo apt update && sudo apt upgrade -y
```

### Bước 2: Cài đặt OpenJDK 17, Gradle, Nginx, MySQL Server, UFW, Unzip, Curl
```bash
sudo apt install -y openjdk-17-jdk gradle nginx mysql-server ufw curl unzip
```

### Bước 3: Cài đặt Node.js và npm (Phiên bản Node 20.x LTS)
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

### Bước 4: Kiểm tra phiên bản các công cụ
```bash
node --version
npm --version
java --version
gradle --version
nginx -v
mysql --version
ufw --version
```
*Kết quả đạt được:*
- `node`: `>= v18.x.x` (hoặc `v20.x.x`)
- `npm`: `>= 9.x.x`
- `java`: `openjdk 17.x.x`
- `gradle`: `Gradle 7.x` hoặc `8.x`
- `nginx`: `nginx/1.18.x` trở lên
- `mysql`: `mysql Ver 8.0.x`
- `ufw`: `ufw 0.36.x`

---

## 1.4. Cấu hình Firewall UFW (10 điểm)

### Bước 1: Thiết lập chính sách Firewall & mở cổng cần thiết
```bash
# Thiết lập mặc định chặn mọi kết nối đến
sudo ufw default deny incoming

# Cho phép mọi kết nối đi ra ngoài
sudo ufw default allow outgoing

# Mở cổng SSH 22
sudo ufw allow 22/tcp

# Chặn trực tiếp truy cập MySQL 3306 từ bên ngoài
sudo ufw deny 3306/tcp
```

### Bước 2: Kích hoạt UFW
```bash
sudo ufw enable
```
*Khi terminal hỏi `Command may disrupt existing ssh connections. Proceed with operation (y|n)?`, gõ `y` rồi nhấn Enter.*

### Bước 3: Kiểm tra trạng thái Firewall
```bash
sudo ufw status numbered
```
*Kết quả đạt được:*
```text
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 3306/tcp                   DENY IN     Anywhere
[ 3] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 4] 3306/tcp (v6)              DENY IN     Anywhere (v6)
```

---

# PHẦN 2: DEPLOY BACKEND & DATABASE (55 ĐIỂM)

## 2.1. Đưa mã nguồn lên VPS & Kiểm tra môi trường ban đầu

### Bước 1: Upload thư mục `devops-hackathon` lên VPS qua Bitvise SSH Client
1. Mở **Bitvise SSH Client** trên máy tính cá nhân.
2. Cấu hình thông tin kết nối:
   - **Host:** `221.121.4.67`
   - **Port:** `22`
   - **Username:** `lplinh-HNK24CNTT1`
   - **Password:** `Devops@2026`
3. Nhấn **Log in**.
4. Nhấn nút **New SFTP Window**:
   - Ở khung bên trái (Local PC): Chọn thư mục dự án `devops-hackathon`.
   - Ở khung bên phải (Remote VPS): Đang ở `/home/lplinh-HNK24CNTT1/`.
   - Kéo thả toàn bộ thư mục `devops-hackathon` sang bên phải.

### Bước 2: Chạy kiểm tra môi trường đầu giờ (`environment-check.js`)
Quay lại Terminal VPS và chạy:
```bash
cd ~/devops-hackathon
node environment-check.js
```
*Kết quả đạt được:* Tool hiển thị danh sách kiểm tra các gói đã cài đặt đạt chuẩn.

---

## 2.2. Cấu hình & Import MySQL Database (15 điểm)

### Bước 1: Mở và chỉnh sửa file `database.sql`
Có thể chỉnh sửa trực tiếp trên IntelliJ rồi upload lên hoặc chỉnh sửa bằng `nano` trên VPS:
```bash
nano ~/devops-hackathon/backend-app/database.sql
```
Thêm/sửa nội dung đầu file như sau:
```sql
-- 1. Tạo database
CREATE DATABASE IF NOT EXISTS task_management_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 2. Tạo User trùng tên tài khoản Linux
CREATE USER IF NOT EXISTS 'lplinh-HNK24CNTT1'@'localhost' IDENTIFIED BY 'Devops@2026';

-- 3. Cấp toàn bộ quyền cho user trên database
GRANT ALL PRIVILEGES ON task_management_db.* TO 'lplinh-HNK24CNTT1'@'localhost';
FLUSH PRIVILEGES;

-- 4. Sử dụng database để tạo bảng và dữ liệu mẫu
USE task_management_db;

-- (Giữ nguyên toàn bộ các lệnh CREATE TABLE / INSERT có sẵn của đề bài bên dưới)
```
*(Bấm `Ctrl + O` -> `Enter` để lưu, `Ctrl + X` để thoát)*

### Bước 2: Import file SQL vào MySQL bằng quyền quản trị
```bash
sudo mysql < ~/devops-hackathon/backend-app/database.sql
```

### Bước 3: Kiểm tra đăng nhập bằng tài khoản sinh viên vừa tạo
```bash
mysql -u lplinh-HNK24CNTT1 -p -e "SHOW DATABASES; USE task_management_db; SHOW TABLES;"
```
- Nhập mật khẩu: `Devops@2026`
*Kết quả đạt được:* Xuất hiện database `task_management_db` cùng danh sách các bảng trong hệ thống.

---

## 2.3. Cấu hình & Build Java Backend bằng Gradle (22 điểm)

### Bước 1: Cấu hình `application.properties`
Mở file `application.properties` trong thư mục `devops-hackathon/backend-app/src/main/resources/application.properties` (hoặc mở trực tiếp trên IntelliJ):
```properties
server.port=8086

spring.datasource.url=jdbc:mysql://localhost:3306/task_management_db?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
spring.datasource.username=lplinh-HNK24CNTT1
spring.datasource.password=Devops@2026
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

### Bước 2: Build project ra file `.jar`
Bạn có 2 lựa chọn (khuyên dùng **Cách 1** chạy trực tiếp trên VPS):

#### Cách 1: Build trực tiếp trên VPS bằng Terminal (Khuyên dùng)
```bash
cd ~/devops-hackathon/backend-app
chmod +x gradlew
./gradlew clean build -x test
```
*(Nếu project không kèm wrapper, dùng lệnh: `gradle clean build -x test`)*

*Kết quả đạt được:*
- Dòng chữ `BUILD SUCCESSFUL` xuất hiện.
- Kiểm tra file jar được sinh ra:
```bash
ls -la ~/devops-hackathon/backend-app/build/libs/
```
*(Ví dụ sinh ra file `app.jar` hoặc `backend-app-0.0.1-SNAPSHOT.jar`)*. Nếu tên file dài, bạn có thể copy hoặc đổi tên thành `app.jar`:
```bash
cp ~/devops-hackathon/backend-app/build/libs/*.jar ~/devops-hackathon/backend-app/build/libs/app.jar
```

#### Cách 2: Build bằng IntelliJ IDEA trên máy cá nhân
1. Mở thư mục `devops-hackathon/backend-app` bằng **IntelliJ IDEA**.
2. Đảm bảo file `application.properties` đã điền đúng thông tin DB.
3. Ở thanh công cụ bên phải, mở tab **Gradle** -> Chọn `Tasks` -> `build` -> Nhấp đúp vào `build`.
4. Tìm file `.jar` trong thư mục `backend-app/build/libs/`.
5. Đổi tên thành `app.jar` và dùng Bitvise SFTP upload vào `/home/lplinh-HNK24CNTT1/devops-hackathon/backend-app/build/libs/app.jar`.

---

## 2.4. Chạy thử nghiệm Backend

### Bước 1: Chạy trực tiếp từ dòng lệnh
```bash
cd ~/devops-hackathon/backend-app
java -jar build/libs/app.jar
```
*Kết quả đạt được:*
- Spring Boot khởi động và xuất hiện thông báo: `Tomcat started on port(s): 8086 (http)` và `Started ... Application in ... seconds`.

### Bước 2: Dừng ứng dụng
- Nhấn tổ hợp phím **`Ctrl + C`** để dừng ứng dụng chạy thủ công (chúng ta sẽ cho chạy bằng Systemd ở bước tiếp theo).

---

## 2.5. Tạo và kích hoạt Systemd Service (13 điểm)

### Bước 1: Tạo file cấu hình dịch vụ Systemd
```bash
sudo nano /etc/systemd/system/lplinh-HNK24CNTT1-task.service
```

Điền chính xác nội dung cấu hình sau:
```ini
[Unit]
Description=PTIT Task Management API
After=network.target mysql.service

[Service]
User=lplinh-HNK24CNTT1
WorkingDirectory=/home/lplinh-HNK24CNTT1/devops-hackathon/backend-app
ExecStart=/usr/bin/java -jar /home/lplinh-HNK24CNTT1/devops-hackathon/backend-app/build/libs/app.jar
Restart=always
RestartSec=5

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```
*(Bấm `Ctrl + O` -> `Enter` để lưu, `Ctrl + X` để thoát)*

### Bước 2: Phân quyền, nạp lại cấu hình và khởi chạy Service
```bash
sudo chmod 644 /etc/systemd/system/lplinh-HNK24CNTT1-task.service
sudo systemctl daemon-reload
sudo systemctl enable lplinh-HNK24CNTT1-task.service
sudo systemctl start lplinh-HNK24CNTT1-task.service
```

### Bước 3: Kiểm tra trạng thái hoạt động của Service
```bash
sudo systemctl status lplinh-HNK24CNTT1-task.service
```
*Kết quả đạt được:*
Hiển thị dòng màu xanh: **`Active: active (running)`**, ứng dụng chạy dưới quyền user `lplinh-HNK24CNTT1`.

---

# PHẦN 3: NGINX REVERSE PROXY & STATIC WEB (10 ĐIỂM)

## 3.1. Tạo Static Website phục vụ tại `/`

### Bước 1: Tạo thư mục và tạo file `index.html`
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
    <title>PTIT DevOps Task Management</title>
</head>
<body>
    <h1>PTIT DevOps Task Management</h1>
    <p>Họ tên: Lê Phương Linh</p>
    <p>Mã sinh viên: HNK24CNTT1</p>
    <p>Lớp: HNK24CNTT1</p>
</body>
</html>
```
*(Bấm `Ctrl + O` -> `Enter` để lưu, `Ctrl + X` để thoát)*

### Bước 2: Phân quyền cho Web Server Nginx truy cập
```bash
sudo chown -R www-data:www-data /var/www/task-app
sudo chmod -R 755 /var/www/task-app
```

---

## 3.2. Cấu hình Nginx Reverse Proxy cổng 8090

### Bước 1: Tạo file cấu hình virtual host Nginx
```bash
sudo nano /etc/nginx/sites-available/lplinh-HNK24CNTT1-task.conf
```

Điền nội dung cấu hình:
```nginx
server {
    listen 8090;
    server_name _;

    # 1. Phục vụ Static Website tại root /
    location / {
        root /var/www/task-app;
        index index.html;
        try_files $uri $uri/ =404;
    }

    # 2. Reverse Proxy đường dẫn /api/ sang Spring Boot Backend (Port 8086)
    location /api/ {
        proxy_pass http://127.0.0.1:8086/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
*(Bấm `Ctrl + O` -> `Enter` để lưu, `Ctrl + X` để thoát)*

### Bước 2: Tạo liên kết kích hoạt (Symlink) và kiểm tra cú pháp Nginx
```bash
sudo ln -s /etc/nginx/sites-available/lplinh-HNK24CNTT1-task.conf /etc/nginx/sites-enabled/
sudo nginx -t
```
*Kết quả đạt được:*
`nginx: configuration file /etc/nginx/nginx.conf test is successful`

### Bước 3: Nạp lại Nginx và mở cổng Firewall 8090
```bash
sudo systemctl reload nginx
sudo ufw allow 8090/tcp
sudo ufw status numbered
```

### Bước 4: Kiểm tra hoạt động
1. Mở trình duyệt trên máy tính cá nhân và truy cập:
   ```text
   http://221.121.4.67:8090/
   ```
   *Kết quả:* Hiển thị trang HTML với tiêu đề "PTIT DevOps Task Management" và thông tin Lê Phương Linh - HNK24CNTT1.
2. Kiểm tra API qua Reverse proxy trên Terminal:
   ```bash
   curl http://127.0.0.1:8090/api/
   ```

---

# PHẦN 4: ĐÓNG GÓI MINH CHỨNG & CHẠY SYSTEM INSPECTOR (10 ĐIỂM)

## 4.1. Chuẩn bị thư mục `submission/`
```bash
cd ~/devops-hackathon
mkdir -p submission
```

## 4.2. Copy các file cấu hình vào `submission/` (3 điểm)
```bash
# Copy file Systemd service
cp /etc/systemd/system/lplinh-HNK24CNTT1-task.service ~/devops-hackathon/submission/

# Copy file Nginx config
cp /etc/nginx/sites-available/lplinh-HNK24CNTT1-task.conf ~/devops-hackathon/submission/
```

## 4.3. Chạy công cụ kiểm tra tự động cuối bài (System Inspector - 4 điểm)
```bash
cd ~/devops-hackathon
node system-inspector.js
```
*Kết quả đạt được:*
Hệ thống kiểm tra toàn diện và thông báo tạo thành công file: `submission/report.enc`.

## 4.4. Lưu lịch sử câu lệnh terminal (History - 3 điểm)
```bash
history > ~/devops-hackathon/submission/history.log
```

---

# PHẦN 5: TẢI BÀI NỘP QUA BITVISE SFTP & CHECKLIST KIỂM TRA

## 5.1. Tải thư mục bài làm về máy cá nhân
1. Mở lại cửa sổ **Bitvise SFTP**.
2. Ở khung bên phải (Remote), mở thư mục `/home/lplinh-HNK24CNTT1/`.
3. Tải toàn bộ thư mục `devops-hackathon` về máy tính của bạn.
4. Kiểm tra cấu trúc thư mục tải về phải đầy đủ như sau:
```text
devops-hackathon/
├── backend-app/
│   ├── build/libs/app.jar
│   ├── src/
│   ├── build.gradle
│   ├── application.properties
│   └── database.sql
├── templates/
│   ├── task-api.service
│   └── task-api.conf
├── submission/
│   ├── history.log
│   ├── lplinh-HNK24CNTT1-task.service
│   ├── lplinh-HNK24CNTT1-task.conf
│   └── report.enc
├── environment-check.js
└── system-inspector.js
```

---

## 5.2. Bảng tổng hợp lệnh kiểm tra nhanh trước khi nộp bài

| STT | Nội dung kiểm tra | Lệnh chạy trên VPS | Kết quả đạt chuẩn |
| :--- | :--- | :--- | :--- |
| **1** | Tài khoản đang thao tác | `whoami` | `lplinh-HNK24CNTT1` (Không phải root) |
| **2** | Quyền Sudo | `sudo -v` | Xác thực thành công |
| **3** | UFW Firewall | `sudo ufw status numbered` | Port 22 ALLOW, Port 8090 ALLOW, Port 3306 DENY |
| **4** | MySQL Database | `mysql -u lplinh-HNK24CNTT1 -p -e "SHOW DATABASES;"` | Có database `task_management_db` |
| **5** | Systemd Service | `sudo systemctl is-active lplinh-HNK24CNTT1-task` | `active` |
| **6** | Nginx Reverse Proxy | `curl -I http://127.0.0.1:8090` | `HTTP/1.1 200 OK` |
| **7** | Minh chứng nộp bài | `ls -la ~/devops-hackathon/submission/` | Đủ 4 file: `history.log`, `report.enc`, `.service`, `.conf` |
