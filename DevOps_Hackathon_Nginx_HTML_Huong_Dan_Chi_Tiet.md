# CẨM NANG TOÀN DIỆN: THỰC HÀNH DEVOPS HACKATHON
## TRIỂN KHAI WEBSITE HTML VỚI NGINX & QUẢN LÝ MÃ NGUỒN QUA GIT TRÊN LINUX VPS

> **BẢNG THÔNG TIN THỰC HÀNH CÁ NHÂN:**
> - **Họ và tên:** Lê Phương Linh
> - **Lớp:** HNK24CNTT1
> - **Tài khoản Linux User:** `lplinh-HNK24CNTT1`
> - **Địa chỉ IP VPS:** `221.121.4.132`
> - **Cổng kết nối SSH:** `22`
> - **Cổng dịch vụ Web Nginx (Custom Port):** `8085` *(hoặc cổng được chỉ định riêng)*
> - **Tên Repository GitHub:** `PTIT_CNTT1_IT209_Demo`

---

# PHẦN I: ĐỀ BÀI PHỤC DỰNG CHUẨN DEVOPS HACKATHON
### ĐỀ SỐ 001: TRIỂN KHAI VÀ QUẢN LÝ WEBSITE HỆ THỐNG PHÒNG LAB VỚI NGINX & GIT

- **Chương trình:** Ptit Rikkei DevOps Lab
- **Đối tượng:** Sinh viên CNTT / DevOps
- **Thời lượng:** 90 - 120 phút
- **Hình thức nộp bài:** GitHub Repository công khai (chứa mã nguồn, file cấu hình Nginx và thư mục ảnh minh chứng `screenshots/`)

---

### 1. Mục tiêu bài thi
Đánh giá năng lực thực hành của thí sinh về:
1. **Quản trị người dùng & Phân quyền Linux:** Tạo user, phân quyền sudo, thiết lập quyền đọc/ghi trên thư mục web (`chown`, `chmod`).
2. **Quản trị mã nguồn với Git & GitHub:** Khởi tạo repository, cấu hình tệp loại trừ `.gitignore` đúng chuẩn DevOps, commit theo từng tính năng độc lập.
3. **Cài đặt & Cấu hình Web Server Nginx:** Viết Virtual Host riêng biệt phục vụ website tĩnh trên cổng tùy chỉnh (Custom Port), cấu hình file log độc lập cho từng sinh viên.
4. **Bảo mật máy chủ với Tường lửa UFW:** Thiết lập chính sách bảo mật, chỉ mở cổng SSH (22) và cổng Web được chỉ định (8085).
5. **Quy trình cập nhật Website thời gian thực (Zero-Downtime Update):** Cập nhật mã nguồn và kéo về máy chủ bằng `git pull` không cần khởi động lại dịch vụ web.
6. **Thu thập minh chứng thực nghiệm:** Chụp ảnh màn hình lưu lại bằng chứng từng giai đoạn theo chuẩn bài thi.

---

### 2. Thông số đề bài quy chuẩn
- **Mã đề:** Đề 001 - *Quản lý phòng Lab*.
- **Hệ điều hành máy chủ:** Ubuntu 22.04 / 24.04 LTS (VPS Cloud).
- **Quy tắc đặt tên Linux User:** `<tên_viết_liền_không_dấu>-<mã_lớp>` *(Ví dụ: `lplinh-HNK24CNTT1`)*.
- **Tên Repository GitHub:** `PTIT_CNTT1_IT209_Demo`.
- **Cổng dịch vụ Nginx:** `8085` (hoặc cổng được chỉ định riêng).
- **Đường dẫn thư mục Web Root:** `/var/www/PTIT_CNTT1_IT209_Demo/src`.

---

### 3. Yêu cầu chi tiết các phần

#### 🔹 Yêu cầu 1: Quản trị Linux User & Môi trường
- Tạo một tài khoản Linux User cục bộ mới theo định dạng: `lplinh-HNK24CNTT1`.
- Cấp quyền quản trị `sudo` cho tài khoản mới.
- Chuyển sang thao tác hoàn toàn dưới quyền của user này.
- Cài đặt đầy đủ các gói: `nginx`, `git`, `ufw`, `curl`.
- Cấu hình định danh Git toàn cục trên máy chủ (`user.name` và `user.email`).

#### 🔹 Yêu cầu 2: Khởi tạo Git & File `.gitignore`
- Tạo repo trên GitHub ở chế độ Public với tên: `PTIT_CNTT1_IT209_Demo`.
- Thiết lập file `.gitignore` ngay tại **commit đầu tiên** của dự án để loại trừ:
  - File nhật ký: `*.log`
  - File hệ điều hành tự sinh: `.DS_Store`, `Thumbs.db`
  - Thư mục môi trường IDE: `.idea/`, `*.iml`, `.vscode/`
  - File tạm thời: `*.tmp`, `*.swp`

#### 🔹 Yêu cầu 3: Xây dựng nội dung Website tĩnh (`src/index.html`)
Trang web phải hiển thị giao diện thông tin chỉn chu với các nội dung bắt buộc:
- Tiêu đề H1: `Xin chào đến với Ptit Rikkei Devops Lab - [Họ tên] - [Mã SV/Lớp]`
- Bảng 1 (Thông tin sinh viên): Họ tên, Mã sinh viên, Lớp, Tài khoản Linux, Link GitHub.
- Bảng 2 (Thông tin bài thi): Số đề (001), Chủ đề (Quản lý phòng Lab), Cổng Nginx (8085), Địa chỉ IP VPS (221.121.4.132), Tên repository (`PTIT_CNTT1_IT209_Demo`).
- Phần giới thiệu: Tóm tắt mục tiêu của hệ thống quản lý phòng Lab.

#### 🔹 Yêu cầu 4: Triển khai Nginx Virtual Host & Phân quyền
- Clone mã nguồn trực tiếp vào thư mục: `/var/www/PTIT_CNTT1_IT209_Demo`.
- Gán quyền sở hữu thư mục web cho tài khoản sinh viên.
- Phân quyền thư mục `755` và quyền tệp tin `644`.
- Viết file cấu hình Virtual Host đặt tại: `nginx/lplinh-HNK24CNTT1.conf`:
  - Lắng nghe cổng `8085` trên cả IPv4 và IPv6.
  - `server_name`: Địa chỉ IP máy chủ (`221.121.4.132`).
  - `root`: Trỏ vào thư mục `src/` của dự án (`/var/www/PTIT_CNTT1_IT209_Demo/src`).
  - `index`: `index.html`.
  - Tách riêng access log và error log: `/var/log/nginx/lplinh-HNK24CNTT1.access.log` và `.error.log`.
  - Phục vụ file tĩnh và điều hướng lỗi với `try_files $uri $uri/ =404;`.
- Liên kết cấu hình vào `/etc/nginx/sites-enabled/`, vô hiệu hóa cấu hình `default`.
- Kiểm tra tính đúng đắn với `nginx -t` và tải lại Nginx.

#### 🔹 Yêu cầu 5: Thiết lập Tường lửa UFW
- Cho phép cổng SSH `22/tcp` để duy trì kết nối quản trị.
- Cho phép cổng Web tùy chỉnh `8085/tcp`.
- Bật tường lửa UFW và đảm bảo trạng thái `active`.

#### 🔹 Yêu cầu 6: Quy trình cập nhật Website (Git Pull Workflow)
- Sửa đổi nội dung file `src/index.html` (thêm thông báo *"Cập nhật lần 2"*).
- Commit và Push lên GitHub.
- Trên VPS, chạy lệnh `git pull` tại `/var/www/PTIT_CNTT1_IT209_Demo`.
- Kiểm tra trên trình duyệt: nội dung mới xuất hiện ngay mà **không cần restart hoặc reload Nginx**.

#### 🔹 Yêu cầu 7: Thu thập 06 ảnh minh chứng (`screenshots/`)
1. `01-user.png`: Lệnh tạo user, cấp sudo và kết quả lệnh `id lplinh-HNK24CNTT1`.
2. `02-nginx.png`: Kết quả kiểm tra `sudo nginx -t` và trạng thái `active (running)` của Nginx.
3. `03-ufw.png`: Trạng thái `sudo ufw status verbose` hiển thị mở cổng 22 và 8085.
4. `04-website.png`: Trình duyệt truy cập thành công `http://221.121.4.132:8085`.
5. `05-git-log.png`: Lịch sử commit trên terminal (`git log --oneline`).
6. `06-update.png`: Trình duyệt hiển thị nội dung website sau khi cập nhật lần 2.

---

# PHẦN II: HƯỚNG DẪN CHI TIẾT TỪNG BƯỚC THỰC HIỆN (A - Z)

---

## BƯỚC 1: KHỞI TẠO DỰ ÁN TRÊN INTELLIJ IDEA & GITHUB

### 1.1. Tạo mới Project trên IntelliJ IDEA
1. Mở phần mềm **IntelliJ IDEA**.
2. Chọn **New Project** (hoặc menu `File` -> `New` -> `Project`).
3. Điền các trường thông tin:
   - **Name:** `PTIT_CNTT1_IT209_Demo`
   - **Location:** Chọn thư mục lưu trên máy bạn (VD: `D:\DevOps - RA\PTIT_CNTT1_IT209_Demo`).
   - Loại dự án: Chọn **Empty Project** hoặc **Static Web**.
4. Nhấn **Create**.

> 🎯 **Kết quả cần đạt được:**
> Cửa sổ IntelliJ IDEA mở ra với dự án mang tên `PTIT_CNTT1_IT209_Demo`. Cây thư mục bên trái sẵn sàng để tạo file.

---

### 1.2. Tạo file `.gitignore` (Loại trừ tệp rác của IntelliJ & Hệ điều hành)
1. Nhấp chuột phải vào thư mục gốc dự án -> chọn **New** -> **File** -> đặt tên chính xác là: `.gitignore`
2. Dán nội dung cấu hình sau vào file:
```gitignore
# Tệp tin nhật ký (Log files)
*.log

# Tệp tin do hệ điều hành tự sinh
.DS_Store
Thumbs.db

# Cấu hình IDE (Đặc biệt quan trọng với IntelliJ IDEA)
.idea/
*.iml
.vscode/

# Tệp tin tạm thời khi soạn thảo
*.tmp
*.swp
```

> 🎯 **Kết quả cần đạt được:**
> File `.gitignore` xuất hiện ở thư mục gốc. Thư mục `.idea/` của IntelliJ sẽ tự động mờ đi (hoặc đổi màu xám), báo hiệu Git sẽ không bao giờ theo dõi thư mục này.

---

### 1.3. Khởi tạo Git cục bộ & Commit lần 1 (`Thêm .gitignore`)
Mở cửa sổ **Terminal** tích hợp trong IntelliJ (phím tắt `Alt + F12`) và chạy:
```bash
git init
git add .gitignore
git commit -m "Thêm .gitignore"
```

> 🎯 **Kết quả cần đạt được:**
> Terminal hiển thị thông báo khởi tạo thành công và commit đầu tiên được ghi nhận:
> ```text
> Initialized empty Git repository in D:/DevOps - RA/PTIT_CNTT1_IT209_Demo/.git/
> [main (root-commit) 7a1b2c3] Thêm .gitignore
>  1 file changed, 16 insertions(+)
>  create mode 100644 .gitignore
> ```

---

### 1.4. Xây dựng trang Web tĩnh `src/index.html`
1. Chuột phải vào thư mục gốc dự án -> chọn **New** -> **Directory** -> đặt tên là: `src`
2. Chuột phải vào thư mục `src` -> chọn **New** -> **HTML File** -> đặt tên là: `index.html`
3. Dán đoạn mã HTML sau:

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DevOps Hackathon - Đề 001 - Lê Phương Linh</title>
    <style>
        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            background-color: #f4f6f9;
            color: #333;
            margin: 0;
            padding: 30px;
        }
        .container {
            max-width: 800px;
            margin: 0 auto;
            background: #ffffff;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.08);
        }
        h1 {
            color: #1a73e8;
            font-size: 24px;
            border-bottom: 2px solid #e8eaed;
            padding-bottom: 12px;
        }
        h2 {
            color: #202124;
            font-size: 18px;
            margin-top: 25px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 10px;
            margin-bottom: 20px;
        }
        th, td {
            border: 1px solid #dadce0;
            padding: 10px 14px;
            text-align: left;
        }
        td:first-child {
            font-weight: 600;
            width: 35%;
            background-color: #f8f9fa;
        }
        .intro-box {
            background-color: #e8f0fe;
            border-left: 4px solid #1a73e8;
            padding: 15px;
            border-radius: 4px;
            line-height: 1.6;
        }
        .footer {
            margin-top: 25px;
            font-size: 13px;
            color: #5f6368;
            text-align: right;
            border-top: 1px dashed #dadce0;
            padding-top: 10px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Xin chào đến với Ptit Rikkei Devops Lab - Lê Phương Linh - HNK24CNTT1</h1>

        <h2>Thông tin sinh viên</h2>
        <table>
            <tr><td>Họ và tên</td><td>Lê Phương Linh</td></tr>
            <tr><td>Mã sinh viên / Lớp</td><td>HNK24CNTT1</td></tr>
            <tr><td>Tài khoản Linux</td><td>lplinh-HNK24CNTT1</td></tr>
            <tr><td>GitHub Profile</td><td><a href="https://github.com/lephuonglinh" target="_blank">lephuonglinh</a></td></tr>
        </table>

        <h2>Thông tin bài thi</h2>
        <table>
            <tr><td>Đề số</td><td>001</td></tr>
            <tr><td>Chủ đề</td><td>Quản lý phòng Lab</td></tr>
            <tr><td>Cổng Nginx (Port)</td><td>8085</td></tr>
            <tr><td>Địa chỉ IP máy chủ</td><td>221.121.4.132</td></tr>
            <tr><td>Repository</td><td>PTIT_CNTT1_IT209_Demo</td></tr>
        </table>

        <h2>Giới thiệu hệ thống</h2>
        <div class="intro-box">
            Hệ thống Quản lý phòng Lab phục vụ quản lý thiết bị, tài nguyên và hỗ trợ sinh viên, giảng viên đăng ký lịch sử dụng phòng máy thực hành một cách thuận tiện, minh bạch.
        </div>

        <div class="footer">
            Phiên bản khởi tạo ban đầu
        </div>
    </div>
</body>
</html>
```

4. Tiến hành commit lần 2 trên Terminal IntelliJ:
```bash
git add src/index.html
git commit -m "Thêm index.html"
```

> 🎯 **Kết quả cần đạt được:**
> Terminal ghi nhận commit thành công:
> ```text
> [main 8b2c3d4] Thêm index.html
>  1 file changed, ... insertions(+)
>  create mode 100644 src/index.html
> ```

---

### 1.5. Tạo file cấu hình Virtual Host Nginx `nginx/lplinh-HNK24CNTT1.conf`
1. Chuột phải vào thư mục gốc dự án -> chọn **New** -> **Directory** -> đặt tên là: `nginx`
2. Chuột phải vào thư mục `nginx` -> chọn **New** -> **File** -> đặt tên là: `lplinh-HNK24CNTT1.conf`
3. Dán toàn bộ nội dung cấu hình sau:

```nginx
# File cấu hình Nginx - DevOps Hackathon Đề 001 - Lê Phương Linh
server {
    listen       8085;
    listen       [::]:8085;

    server_name  221.121.4.132;

    # Đường dẫn Web Root trỏ vào thư mục src của dự án trên VPS
    root         /var/www/PTIT_CNTT1_IT209_Demo/src;
    index        index.html;

    # Tách file log riêng biệt theo tên sinh viên
    access_log   /var/log/nginx/lplinh-HNK24CNTT1.access.log;
    error_log    /var/log/nginx/lplinh-HNK24CNTT1.error.log;

    location / {
        allow all;
        try_files $uri $uri/ =404;
    }
}
```

4. Thực hiện commit lần 3:
```bash
git add nginx/lplinh-HNK24CNTT1.conf
git commit -m "Thêm file cấu hình nginx"
```

> 🎯 **Kết quả cần đạt được:**
> Terminal xác nhận commit thành công file cấu hình nginx:
> ```text
> [main 9c3d4e5] Thêm file cấu hình nginx
>  1 file changed, 19 insertions(+)
>  create mode 100644 nginx/lplinh-HNK24CNTT1.conf
> ```

---

### 1.6. Tạo thư mục `screenshots/`
Chuột phải vào thư mục gốc dự án -> chọn **New** -> **Directory** -> đặt tên là: `screenshots`

> 🎯 **Kết quả cần đạt được:**
> Thư mục `screenshots` xuất hiện trong cấu trúc dự án trên IntelliJ, sẵn sàng để sau này chứa 6 ảnh minh chứng.

---

### 1.7. Đẩy mã nguồn lên GitHub
1. Mở trình duyệt web, đăng nhập vào GitHub của bạn và bấm **New repository**.
2. Nhập Repository name: `PTIT_CNTT1_IT209_Demo`
   - Đặt chế độ: **Public**.
   - **LƯU Ý:** Không tích vào bất kỳ ô nào (*Add a README file*, *Add .gitignore*).
   - Nhấn nút **Create repository**.
3. Copy đường dẫn Git (dạng: `https://github.com/<username-cua-ban>/PTIT_CNTT1_IT209_Demo.git`).
4. Mở lại Terminal trên IntelliJ và chạy:
```bash
git branch -M main
git remote add origin https://github.com/<username-cua-ban>/PTIT_CNTT1_IT209_Demo.git
git push -u origin main
```

> 🎯 **Kết quả cần đạt được:**
> Terminal báo đẩy code thành công:
> ```text
> To https://github.com/<username-cua-ban>/PTIT_CNTT1_IT209_Demo.git
>  * [new branch]      main -> main
> branch 'main' set up to track 'origin/main'.
> ```
> Khi F5 trang GitHub trên trình duyệt, bạn thấy repository có 3 commit sạch sẽ, chứa đúng 3 thành phần: `.gitignore`, `src/index.html` và `nginx/lplinh-HNK24CNTT1.conf`.

---

## BƯỚC 2: KẾT NỐI VÀ THIẾT LẬP MÁY CHỦ LINUX VPS

### 2.1. Đăng nhập vào VPS qua SSH
Mở PowerShell (hoặc Bitvise SSH Client) trên máy tính:
```powershell
ssh root@221.121.4.132
```
Nhập mật khẩu `root` khi được hỏi.

> 💡 **Khắc phục lỗi nếu VPS vừa Reset OS:**
> Nếu báo lỗi `Host key verification failed`, hãy chạy lệnh: `ssh-keygen -R 221.121.4.132` rồi ssh lại.

> 🎯 **Kết quả cần đạt được:**
> Đăng nhập thành công vào VPS, màn hình hiển thị dấu nhắc lệnh: `root@...:~#`.

---

### 2.2. Tạo tài khoản User `lplinh-HNK24CNTT1` & Cấp quyền Sudo
Chạy các lệnh sau khi đang ở quyền `root`:
```bash
# 1. Tạo user mới có thư mục home và shell bash
sudo useradd -m -s /bin/bash lplinh-HNK24CNTT1

# 2. Cấp quyền sudo quản trị cho user
sudo usermod -aG sudo lplinh-HNK24CNTT1
```

> 🔑 **Tùy chọn Mật khẩu cho User:**
> - **Cách A (Khuyên dùng - Đặt pass siêu ngắn):** Chạy `sudo passwd lplinh-HNK24CNTT1` và nhập `123456` (2 lần).
> - **Cách B (Không cần mật khẩu):** Chạy lệnh:
>   `echo "lplinh-HNK24CNTT1 ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/lplinh-HNK24CNTT1`
>   *(Lệnh này giúp sau này khi gõ `sudo` hệ thống không bao giờ đòi hỏi mật khẩu).*

> 🎯 **Kết quả cần đạt được:**
> User được tạo thành công, không có thông báo lỗi phát sinh.

---

### 2.3. Chuyển sang User mới & Kiểm tra phân quyền
```bash
# Chuyển sang user sinh viên
su - lplinh-HNK24CNTT1

# Kiểm tra phân quyền user
id lplinh-HNK24CNTT1
```

> 🎯 **Kết quả cần đạt được:**
> 1. Dấu nhắc lệnh đổi thành: `lplinh-HNK24CNTT1@...:~$`
> 2. Màn hình in ra thông tin phân quyền chứa nhóm `sudo`:
>    ```text
>    uid=1001(lplinh-HNK24CNTT1) gid=1001(lplinh-HNK24CNTT1) groups=1001(lplinh-HNK24CNTT1),27(sudo)
>    ```

📸 **CHỤP ẢNH MINH CHỨNG SỐ 1:**
Chụp toàn bộ cửa sổ terminal hiển thị lệnh `id` và dấu nhắc lệnh mang tên user của bạn -> Lưu tên file là: **`01-user.png`**.

---

### 2.4. Cập nhật hệ thống & Cài đặt phần mềm cần thiết
Chạy lệnh cài đặt các gói công cụ:
```bash
sudo apt update -y
sudo apt install -y nginx git ufw curl
sudo systemctl enable --now nginx
```

> 🎯 **Kết quả cần đạt được:**
> Các gói phần mềm được cài đặt đầy đủ. Dịch vụ Nginx được bật tự khởi động cùng hệ thống (`Created symlink ... /etc/systemd/system/multi-user.target.wants/nginx.service`).

---

### 2.5. Cấu hình thông tin Git cá nhân trên VPS
```bash
git config --global user.name "Le Phuong Linh"
git config --global user.email "linhlp@ptit.edu.vn"
```

> 🎯 **Kết quả cần đạt được:**
> Chạy lệnh `git config --list` sẽ thấy xuất hiện 2 dòng `user.name=Le Phuong Linh` và `user.email=linhlp@ptit.edu.vn`.

---

## BƯỚC 3: DEPLOY DỰ ÁN TỪ GITHUB VÀO `/var/www/`

### 3.1. Clone mã nguồn từ GitHub vào thư mục Web
Thực hiện lệnh clone vào `/var/www/PTIT_CNTT1_IT209_Demo`:
```bash
sudo git clone https://github.com/<username-cua-ban>/PTIT_CNTT1_IT209_Demo.git /var/www/PTIT_CNTT1_IT209_Demo
```

> 🎯 **Kết quả cần đạt được:**
> Terminal thông báo tải mã nguồn thành công:
> ```text
> Cloning into '/var/www/PTIT_CNTT1_IT209_Demo'...
> Receiving objects: 100% (...), done.
> ```

---

### 3.2. Phân quyền sở hữu & Cấp quyền truy cập chuẩn
```bash
# 1. Gán quyền sở hữu thư mục dự án cho user sinh viên
sudo chown -R lplinh-HNK24CNTT1:lplinh-HNK24CNTT1 /var/www/PTIT_CNTT1_IT209_Demo

# 2. Cấp quyền 755 cho toàn bộ thư mục (để Nginx www-data đọc được)
find /var/www/PTIT_CNTT1_IT209_Demo -type d -exec chmod 755 {} \;

# 3. Cấp quyền 644 cho toàn bộ tệp tin
find /var/www/PTIT_CNTT1_IT209_Demo -type f -exec chmod 644 {} \;
```

> 🎯 **Kết quả cần đạt được:**
> Kiểm tra bằng lệnh `ls -ld /var/www/PTIT_CNTT1_IT209_Demo` sẽ thấy quyền hiển thị chính xác:
> ```text
> drwxr-xr-x ... lplinh-HNK24CNTT1 lplinh-HNK24CNTT1 ... /var/www/PTIT_CNTT1_IT209_Demo
> ```

---

## BƯỚC 4: KÍCH HOẠT VÀ VẬN HÀNH NGINX VIRTUAL HOST

### 4.1. Kích hoạt Virtual Host Nginx
```bash
# 1. Copy file cấu hình từ repo sang sites-available
sudo cp /var/www/PTIT_CNTT1_IT209_Demo/nginx/lplinh-HNK24CNTT1.conf /etc/nginx/sites-available/

# 2. Tạo liên kết tượng trưng (Symlink) sang sites-enabled
sudo ln -s /etc/nginx/sites-available/lplinh-HNK24CNTT1.conf /etc/nginx/sites-enabled/

# 3. Xóa cấu hình mặc định (default) để tránh xung đột
sudo rm -f /etc/nginx/sites-enabled/default
```

> 🎯 **Kết quả cần đạt được:**
> Kiểm tra bằng lệnh: `ls -l /etc/nginx/sites-enabled/` sẽ thấy liên kết trỏ đúng:
> ```text
> lrwxrwxrwx ... lplinh-HNK24CNTT1.conf -> /etc/nginx/sites-available/lplinh-HNK24CNTT1.conf
> ```

---

### 4.2. Kiểm tra cú pháp và nạp lại cấu hình Nginx
```bash
# 1. Kiểm tra tính hợp lệ của file cấu hình
sudo nginx -t

# 2. Tải lại Nginx
sudo systemctl reload nginx

# 3. Kiểm tra trạng thái hoạt động của Nginx
sudo systemctl status nginx
```

> 🎯 **Kết quả cần đạt được:**
> 1. Lệnh `sudo nginx -t` in ra:
>    ```text
>    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
>    nginx: configuration file /etc/nginx/nginx.conf test is successful
>    ```
> 2. Trạng thái dịch vụ hiển thị chữ màu xanh lá: **`Active: active (running)`**.

📸 **CHỤP ẢNH MINH CHỨNG SỐ 2:**
Chụp lại màn hình terminal hiển thị cả kết quả lệnh `sudo nginx -t` và `Active: active (running)` của lệnh status -> Lưu tên file là: **`02-nginx.png`**. *(Nhấn phím `q` để thoát màn hình status)*.

---

## BƯỚC 5: CẤU HÌNH TƯỜNG LỬA BẢO MẬT (UFW FIREWALL)

### 5.1. Thiết lập các quy tắc cho UFW
```bash
# 1. Mở cổng SSH (bắt buộc để không mất kết nối VPS)
sudo ufw allow 22/tcp

# 2. Mở cổng phục vụ Web Nginx (8085)
sudo ufw allow 8085/tcp

# 3. Bật tường lửa (nhập 'y' và ấn Enter khi terminal hỏi)
sudo ufw enable

# 4. Kiểm tra trạng thái chi tiết của tường lửa
sudo ufw status verbose
```

> 🎯 **Kết quả cần đạt được:**
> Màn hình hiển thị bảng tường lửa với trạng thái **active** và mở 2 cổng 22, 8085:
> ```text
> Status: active
> Logging: on (low)
> Default: deny (incoming), allow (outgoing), disabled (routed)
> New profiles: skip
> 
> To                         Action      From
> --                         ------      ----
> 22/tcp                     ALLOW IN    Anywhere
> 8085/tcp                   ALLOW IN    Anywhere
> 22/tcp (v6)                ALLOW IN    Anywhere (v6)
> 8085/tcp (v6)              ALLOW IN    Anywhere (v6)
> ```

📸 **CHỤP ẢNH MINH CHỨNG SỐ 3:**
Chụp toàn bộ kết quả của lệnh `sudo ufw status verbose` trên terminal -> Lưu tên file là: **`03-ufw.png`**.

---

### 5.2. Kiểm tra phản hồi HTTP bằng lệnh Curl trên VPS
Chạy lệnh curl kiểm tra trực tiếp trên máy chủ:
```bash
curl -I http://221.121.4.132:8085
```

> 🎯 **Kết quả cần đạt được:**
> Server phản hồi mã trạng thái HTTP 200:
> ```text
> HTTP/1.1 200 OK
> Server: nginx/...
> Content-Type: text/html
> ...
> ```

---

## BƯỚC 6: KIỂM TRA TRUY CẬP TRÊN TRÌNH DUYỆT & LỊCH SỬ GIT

### 6.1. Kiểm tra Website trên trình duyệt cá nhân
Mở trình duyệt (Chrome, Edge hoặc Firefox) trên máy tính cá nhân và truy cập:
```text
http://221.121.4.132:8085
```

> 🎯 **Kết quả cần đạt được:**
> Trang web tải thành công, hiển thị giao diện đẹp mắt với đầy đủ thông tin:
> - Tiêu đề: *Xin chào đến với Ptit Rikkei Devops Lab - Lê Phương Linh - HNK24CNTT1*
> - Hai bảng thông tin sinh viên và thông tin bài thi.

📸 **CHỤP ẢNH MINH CHỨNG SỐ 4:**
Chụp toàn bộ cửa sổ trình duyệt (nhìn thấy rõ URL `http://221.121.4.132:8085` trên thanh địa chỉ và nội dung website) -> Lưu tên file là: **`04-website.png`**.

---

### 6.2. Kiểm tra lịch sử Git trên máy chủ VPS
Trên cửa sổ Terminal của VPS, chuyển vào thư mục dự án và gõ lệnh xem lịch sử:
```bash
cd /var/www/PTIT_CNTT1_IT209_Demo
git log --oneline
```

> 🎯 **Kết quả cần đạt được:**
> Terminal hiển thị 3 commit ban đầu:
> ```text
> 9c3d4e5 Thêm file cấu hình nginx
> 8b2c3d4 Thêm index.html
> 7a1b2c3 Thêm .gitignore
> ```

📸 **CHỤP ẢNH MINH CHỨNG SỐ 5:**
Chụp lại màn hình terminal hiển thị kết quả của lệnh `git log --oneline` -> Lưu tên file là: **`05-git-log.png`**.

---

## BƯỚC 7: MÔ PHỎNG QUY TRÌNH CẬP NHẬT WEBSITE (GIT PULL WORKFLOW)

Đây là bước thực hành quy trình cập nhật website không gián đoạn dịch vụ:

### 7.1. Chỉnh sửa nội dung file trên máy tính (IntelliJ IDEA)
1. Mở file `src/index.html` trong dự án trên **IntelliJ IDEA**.
2. Tìm đến phần `<div class="footer">` ở cuối file và sửa thành:
```html
        <div class="footer">
            <p><strong>Cập nhật lần 2</strong> - Triển khai hoàn tất Nginx & UFW trên VPS thành công!</p>
        </div>
```
3. Lưu file lại (`Ctrl + S`).

> 🎯 **Kết quả cần đạt được:**
> File `src/index.html` được ghi nhận đã thay đổi (chữ đổi màu trên cây thư mục IntelliJ).

---

### 7.2. Commit và Push mã nguồn cập nhật lên GitHub
Mở Terminal trên IntelliJ IDEA và chạy:
```bash
git add src/index.html
git commit -m "Cập nhật lần 2"
git push origin main
```

> 🎯 **Kết quả cần đạt được:**
> Code mới được đẩy thành công lên nhánh `main` của GitHub.

---

### 7.3. Kéo mã nguồn mới về máy chủ VPS bằng `git pull`
Quay lại cửa sổ Terminal của VPS (đang ở thư mục `/var/www/PTIT_CNTT1_IT209_Demo`), chạy lệnh:
```bash
git pull
```

> 🎯 **Kết quả cần đạt được:**
> Git thông báo cập nhật thành công:
> ```text
> Updating 9c3d4e5..d4e5f6a
> Fast-forward
>  src/index.html | 2 +-
>  1 file changed, 1 insertion(+), 1 deletion(-)
> ```
> *(Lưu ý: Không cần chạy bất kỳ lệnh khởi động lại Nginx nào).*

---

### 7.4. Kiểm tra trên trình duyệt
Quay lại trình duyệt web và nhấn phím **F5** (hoặc `Ctrl + F5`) để tải lại trang `http://221.121.4.132:8085`.

> 🎯 **Kết quả cần đạt được:**
> Trang web hiển thị ngay lập tức dòng chữ: *"Cập nhật lần 2 - Triển khai hoàn tất Nginx & UFW trên VPS thành công!"*.

📸 **CHỤP ẢNH MINH CHỨNG SỐ 6:**
Chụp lại toàn bộ màn hình trình duyệt có dòng chữ *"Cập nhật lần 2"* -> Lưu tên file là: **`06-update.png`**.

---

## BƯỚC 8: ĐẨY TOÀN BỘ ẢNH MINH CHỨNG LÊN GITHUB & HOÀN TẤT BÀI THI

### 8.1. Đưa 6 ảnh chụp vào thư mục `screenshots/` trên IntelliJ
1. Copy cả 6 ảnh chụp màn hình từ máy tính vào thư mục `screenshots/` trong dự án IntelliJ IDEA:
   - `01-user.png`
   - `02-nginx.png`
   - `03-ufw.png`
   - `04-website.png`
   - `05-git-log.png`
   - `06-update.png`

> 🎯 **Kết quả cần đạt được:**
> Thư mục `screenshots/` trên IntelliJ hiển thị đủ 6 tệp tin ảnh PNG.

---

### 8.2. Commit và Push ảnh minh chứng lên GitHub
Chạy các lệnh sau trên Terminal của IntelliJ IDEA:
```bash
git add screenshots/
git commit -m "Thêm ảnh minh chứng"
git push origin main
```

> 🎯 **Kết quả cần đạt được:**
> Toàn bộ ảnh được đẩy lên GitHub thành công.
> Khi mở giao diện GitHub Repository trên trình duyệt, lịch sử commit hiển thị đầy đủ và chuyên nghiệp:
> ```text
> - Thêm ảnh minh chứng
> - Cập nhật lần 2
> - Thêm file cấu hình nginx
> - Thêm index.html
> - Thêm .gitignore
> ```

---

## BẢNG TỔNG HỢP KIỂM TRA NHANH TRƯỚC KHI NỘP BÀI

| Hạng mục kiểm tra | Lệnh thực thi trên VPS | Dấu hiệu đạt chuẩn tuyệt đối |
| :--- | :--- | :--- |
| **1. User thực thi** | `whoami` | Hiển thị: `lplinh-HNK24CNTT1` |
| **2. Quyền thư mục Web** | `ls -ld /var/www/PTIT_CNTT1_IT209_Demo` | Chủ sở hữu là `lplinh-HNK24CNTT1 lplinh-HNK24CNTT1`, quyền `drwxr-xr-x` |
| **3. Cú pháp Nginx** | `sudo nginx -t` | Báo `syntax is ok` & `test is successful` |
| **4. Trạng thái Nginx** | `sudo systemctl is-active nginx` | Báo `active` |
| **5. Trạng thái UFW** | `sudo ufw status numbered` | Cổng `22/tcp` và `8085/tcp` đều ở trạng thái `ALLOW IN` |
| **6. Phản hồi Server** | `curl -I http://221.121.4.132:8085` | Trả về `HTTP/1.1 200 OK` |
| **7. Repository GitHub** | Mở trên trình duyệt web | Đầy đủ thư mục `src/`, `nginx/`, `.gitignore` và `screenshots/` (đủ 6 ảnh) |
