# CẨM NANG TOÀN DIỆN: THỰC HÀNH DEVOPS HACKATHON
## TRIỂN KHAI WEBSITE HTML VỚI NGINX & QUẢN LÝ MÃ NGUỒN QUA GIT TRÊN LINUX VPS

> **BẢNG THÔNG TIN CẤU HÌNH THỰC HÀNH:**
> - **Họ và tên:** Lê Phương Linh
> - **Lớp:** HNK24CNTT1
> - **Tài khoản Linux User:** `lplinh-HNK24CNTT1`
> - **Địa chỉ IP VPS:** `221.121.4.67`
> - **Cổng kết nối SSH:** `22`
> - **Cổng dịch vụ Web Nginx (Custom Port):** `8085` *(hoặc cổng theo bạn chỉ định)*
> - **Tên Repository GitHub:** `devops-hackathon-de001-lephuonglinh`

---

# PHẦN I: ĐỀ BÀI CHUẨN DEVOPS HACKATHON (ĐỀ SỐ 001)

### 1. Mục tiêu bài thi
Đánh giá kỹ năng toàn diện về:
- Quản trị hệ thống máy chủ Linux Ubuntu (Tạo user, phân quyền sudo, quản lý tiến trình).
- Thiết lập bảo mật mạng với tường lửa **UFW**.
- Cài đặt, viết file cấu hình Virtual Host và vận hành Web Server **Nginx**.
- Quản trị mã nguồn chuyên nghiệp với **Git & GitHub** (tạo `.gitignore`, commit phân tách, push code).
- Quy trình cập nhật website thời gian thực không gián đoạn dịch vụ (*zero-downtime update* bằng `git pull` trực tiếp trên máy chủ).

---

### 2. Thông số bài làm quy chuẩn
- **Mã đề:** Đề 001 - *Quản lý phòng Lab*.
- **Hệ điều hành máy chủ:** Ubuntu 22.04 / 24.04 LTS (VPS Cloud).
- **Tài khoản Linux User:** `<tên_viết_liền_không_dấu>-<mã_lớp>` *(VD: `lplinh-HNK24CNTT1`)*.
- **Tên Repo GitHub:** `devops-hackathon-de001-<họ_tên>` *(VD: `devops-hackathon-de001-lephuonglinh`)*.
- **Cổng Nginx phục vụ website:** `8085` (hoặc cổng được chỉ định riêng).
- **Đường dẫn Web Root trên máy chủ:** `/var/www/devops-hackathon-de001-<họ_tên>/src`.

---

### 3. Cấu trúc thư mục dự án quy chuẩn
```text
devops-hackathon-de001-lephuonglinh/
├── .gitignore                      <-- File loại trừ tệp rác (commit đầu tiên)
├── src/
│   └── index.html                  <-- Trang web tĩnh chứa thông tin sinh viên & đề thi
├── nginx/
│   └── lplinh-HNK24CNTT1.conf      <-- File cấu hình Virtual Host Nginx độc lập
└── screenshots/                    <-- Thư mục chứa 6 ảnh minh chứng thực nghiệm
    ├── 01-user.png
    ├── 02-nginx.png
    ├── 03-ufw.png
    ├── 04-website.png
    ├── 05-git-log.png
    └── 06-update.png
```

---

### 4. Tiêu chí đánh giá & Bảng điểm (Thang điểm 10)

| STT | Tiêu chí đánh giá | Yêu cầu kỹ thuật | Điểm |
| :---: | :--- | :--- | :---: |
| **1** | **Khởi tạo Git & `.gitignore`** | Repo sạch, commit đầu tiên có file `.gitignore` loại trừ IDE, OS, Log | 1.0 |
| **2** | **Quản trị Linux User** | Tạo đúng user `lplinh-HNK24CNTT1`, cấp sudo, chạy lệnh qua user này | 1.0 |
| **3** | **Cài đặt & Cấu hình Nginx** | File config riêng trong `nginx/`, port 8085, tách file log riêng, symlink chuẩn | 2.5 |
| **4** | **Bảo mật Tường lửa UFW** | Kích hoạt UFW, mở đúng port 22 và port 8085, chặn các truy cập khác | 1.5 |
| **5** | **Giao diện Website HTML** | Hiển thị đầy đủ thông tin cá nhân, đề thi, bảng biểu rõ ràng | 1.5 |
| **6** | **Quy trình Git Workflow** | Clone về `/var/www/`, cập nhật nội dung lần 2 bằng `git pull` không cần restart Nginx | 1.0 |
| **7** | **Minh chứng Screenshots** | Đầy đủ 6 ảnh minh chứng từ `01-user.png` đến `06-update.png` | 1.5 |
| **TỔNG** | | | **10.0** |

---

# PHẦN II: HƯỚNG DẪN CHI TIẾT CÁCH LÀM TỪNG BƯỚC (A - Z)

---

## BƯỚC 1: KHỞI TẠO DỰ ÁN TRÊN INTELLIJ IDEA & GITHUB

### 1.1. Tạo mới Project trên IntelliJ IDEA
1. Mở phần mềm **IntelliJ IDEA**.
2. Chọn **New Project** (hoặc `File` -> `New` -> `Project`).
3. Điền các thông tin:
   - **Name:** `devops-hackathon-de001-lephuonglinh`
   - **Location:** Chọn thư mục lưu trữ trên máy tính của bạn (VD: `D:\DevOps - RA\devops-hackathon-de001-lephuonglinh`).
   - Loại dự án: Chọn **Empty Project** (hoặc **Static Web / HTML**).
   - Bấm **Create**.

---

### 1.2. Tạo file `.gitignore` (Bắt buộc cho dự án dùng IntelliJ)
Vì bạn dùng IntelliJ IDEA, phần mềm sẽ tự tạo thư mục `.idea/` và các file `*.iml`. File `.gitignore` sẽ đảm bảo các file này không bị đẩy lên GitHub.

1. Tại cửa sổ Project bên trái trong IntelliJ, nhấp chuột phải vào thư mục gốc dự án -> chọn **New** -> **File** -> đặt tên là: `.gitignore`
2. Dán nội dung chuẩn sau vào file:
```gitignore
# Nhật ký hệ thống (Log files)
*.log

# Tệp tin do hệ điều hành tự sinh
.DS_Store
Thumbs.db

# Cấu hình IDE (IntelliJ IDEA & VS Code)
.idea/
*.iml
.vscode/

# Tệp tin tạm thời khi soạn thảo
*.tmp
*.swp
```

---

### 1.3. Khởi tạo Git cục bộ & Commit lần 1 (`Thêm .gitignore`)
Mở cửa sổ **Terminal** ngay bên dưới của IntelliJ IDEA (phím tắt `Alt + F12`) và chạy các lệnh:
```bash
git init
git add .gitignore
git commit -m "Thêm .gitignore"
```

---

### 1.4. Tạo trang Web tĩnh `src/index.html`
1. Chuột phải vào thư mục gốc dự án -> **New** -> **Directory** -> đặt tên là `src`.
2. Chuột phải vào thư mục `src` vừa tạo -> **New** -> **HTML File** (hoặc **File**) -> đặt tên là `index.html`.
3. Dán đoạn mã HTML sau (đã định dạng thẩm mỹ, chứa đầy đủ bảng thông tin cá nhân và đề thi):

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DevOps Hackathon - Đề 001 - Lê Phương Linh</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f2f5;
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
            <tr><td>Địa chỉ IP máy chủ</td><td>221.121.4.67</td></tr>
            <tr><td>Repository</td><td>devops-hackathon-de001-lephuonglinh</td></tr>
        </table>

        <h2>Giới thiệu hệ thống</h2>
        <div class="intro-box">
            Hệ thống Quản lý phòng Lab phục vụ việc đăng ký và quản lý thiết bị thực hành tại các phòng máy. Sinh viên và giảng viên có thể theo dõi lịch biểu, trạng thái máy trạm và tài nguyên thực hành theo thời gian thực.
        </div>

        <div class="footer">
            Phiên bản khởi tạo ban đầu
        </div>
    </div>
</body>
</html>
```

4. Tiến hành commit lần 2 trên Terminal của IntelliJ:
```bash
git add src/index.html
git commit -m "Thêm index.html"
```

---

### 1.5. Tạo file cấu hình Nginx `nginx/lplinh-HNK24CNTT1.conf`
1. Chuột phải vào thư mục gốc dự án -> **New** -> **Directory** -> đặt tên là `nginx`.
2. Chuột phải vào thư mục `nginx` -> **New** -> **File** -> đặt tên là `lplinh-HNK24CNTT1.conf`.
3. Dán nội dung cấu hình Virtual Host Nginx sau:

```nginx
# File cấu hình Nginx cho DevOps Hackathon - Đề 001 - Le Phuong Linh
server {
    listen       8085;
    listen       [::]:8085;

    server_name  221.121.4.67;

    # Trỏ đến thư mục src của dự án được clone trên VPS
    root         /var/www/devops-hackathon-de001-lephuonglinh/src;
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

4. Tiến hành commit lần 3:
```bash
git add nginx/lplinh-HNK24CNTT1.conf
git commit -m "Thêm file cấu hình nginx"
```

---

### 1.6. Tạo thư mục `screenshots/`
Chuột phải vào thư mục gốc dự án -> **New** -> **Directory** -> đặt tên là `screenshots`.
*(Thư mục này sẽ dùng để lưu 6 ảnh chụp màn hình trong quá trình làm bài).*

---

### 1.7. Đẩy mã nguồn lên GitHub
1. Mở trình duyệt web, đăng nhập vào GitHub của bạn và bấm **New Repository**.
2. Đặt tên repository: `devops-hackathon-de001-lephuonglinh`
   - Chọn chế độ: **Public**.
   - **LƯU Ý:** Không tích chọn *Add a README file*, *Add .gitignore* (vì máy tính đã có rồi).
   - Nhấn **Create repository**.
3. Copy link git của repo vừa tạo (dạng: `https://github.com/<username-cua-ban>/devops-hackathon-de001-lephuonglinh.git`).
4. Quay lại Terminal trên IntelliJ IDEA và chạy lệnh push:
```bash
git branch -M main
git remote add origin https://github.com/<username-cua-ban>/devops-hackathon-de001-lephuonglinh.git
git push -u origin main
```

---

## BƯỚC 2: KẾT NỐI VÀ QUẢN TRỊ LINUX TRÊN VPS

### 2.1. Đăng nhập vào VPS qua SSH (hoặc Bitvise)
Mở PowerShell (hoặc Terminal trên máy tính):
```powershell
ssh root@221.121.4.67
```
*(Nhập mật khẩu VPS được cấp để đăng nhập).*

> ⚠️ **Nếu gặp lỗi `Host key verification failed` (khi VPS vừa được reset lại OS):**
> Chạy lệnh sau trên máy tính để xóa khóa cũ rồi đăng nhập lại:
> ```powershell
> ssh-keygen -R 221.121.4.67
> ```

---

### 2.2. Tạo tài khoản User sinh viên `lplinh-HNK24CNTT1` & Cấp quyền Sudo
Khi đang ở quyền `root` trên VPS, gõ:
```bash
# 1. Tạo user mới có thư mục home và shell bash
sudo useradd -m -s /bin/bash lplinh-HNK24CNTT1

# 2. Đặt mật khẩu cho user mới (Nhập ví dụ: 123456789)
sudo passwd lplinh-HNK24CNTT1

# 3. Cấp quyền sudo quản trị cho user
sudo usermod -aG sudo lplinh-HNK24CNTT1
```

---

### 2.3. Chuyển sang User mới & Kiểm tra phân quyền
```bash
# Chuyển sang user sinh viên
su - lplinh-HNK24CNTT1

# Kiểm tra phân quyền
id lplinh-HNK24CNTT1
```
*Kết quả hiển thị trên màn hình tương tự:*
`uid=1001(lplinh-HNK24CNTT1) gid=1001(lplinh-HNK24CNTT1) groups=1001(lplinh-HNK24CNTT1),27(sudo)`

📸 **CHỤP ẢNH MINH CHỨNG SỐ 1:**
Chụp lại màn hình terminal chứa lệnh `id` và dấu nhắc lệnh `lplinh-HNK24CNTT1@...` -> Lưu tên file là: **`01-user.png`**.

---

### 2.4. Cài đặt các gói phần mềm cần thiết
Chạy lệnh cài đặt Nginx, Git, UFW và Curl:
```bash
sudo apt update -y
sudo apt install -y nginx git ufw curl
sudo systemctl enable --now nginx
```

---

### 2.5. Cấu hình thông tin Git cá nhân trên VPS
```bash
git config --global user.name "Le Phuong Linh"
git config --global user.email "linhlp@ptit.edu.vn"
```

---

## BƯỚC 3: DEPLOY MÃ NGUỒN TỪ GITHUB VÀO `/var/www/`

### 3.1. Clone mã nguồn từ GitHub về thư mục Web
Thực hiện lệnh clone trực tiếp vào thư mục `/var/www/`:
```bash
sudo git clone https://github.com/<username-cua-ban>/devops-hackathon-de001-lephuonglinh.git /var/www/devops-hackathon-de001-lephuonglinh
```

### 3.2. Phân quyền sở hữu và cấp quyền truy cập chuẩn
```bash
# Gán quyền sở hữu thư mục dự án cho user sinh viên
sudo chown -R lplinh-HNK24CNTT1:lplinh-HNK24CNTT1 /var/www/devops-hackathon-de001-lephuonglinh

# Cấp quyền 755 cho thư mục (cho phép Nginx www-data đọc và truy cập)
find /var/www/devops-hackathon-de001-lephuonglinh -type d -exec chmod 755 {} \;

# Cấp quyền 644 cho file
find /var/www/devops-hackathon-de001-lephuonglinh -type f -exec chmod 644 {} \;
```

---

## BƯỚC 4: KÍCH HOẠT VÀ VẬN HÀNH NGINX VIRTUAL HOST

### 4.1. Kích hoạt Virtual Host Nginx
Tạo liên kết cấu hình từ dự án vào hệ thống Nginx:
```bash
# Copy hoặc symlink file cấu hình vào sites-available
sudo cp /var/www/devops-hackathon-de001-lephuonglinh/nginx/lplinh-HNK24CNTT1.conf /etc/nginx/sites-available/

# Tạo liên kết tượng trưng (symlink) sang sites-enabled
sudo ln -s /etc/nginx/sites-available/lplinh-HNK24CNTT1.conf /etc/nginx/sites-enabled/

# Gỡ bỏ trang mặc định của Nginx để tránh xung đột
sudo rm -f /etc/nginx/sites-enabled/default
```

### 4.2. Kiểm tra cú pháp và tải lại Nginx
```bash
# 1. Kiểm tra tính hợp lệ của cấu hình
sudo nginx -t

# 2. Reload Nginx để áp dụng cấu hình mới
sudo systemctl reload nginx

# 3. Xem trạng thái dịch vụ Nginx
sudo systemctl status nginx
```
*Kết quả:* Hệ thống báo `syntax is ok` và trạng thái `Active: active (running)`.

📸 **CHỤP ẢNH MINH CHỨNG SỐ 2:**
Chụp lại màn hình terminal hiển thị `sudo nginx -t` thành công và `Active: active (running)` của lệnh status -> Lưu file là: **`02-nginx.png`**.

---

## BƯỚC 5: CẤU HÌNH TƯỜNG LỬA BẢO MẬT (UFW FIREWALL)

### 5.1. Thiết lập các quy tắc tường lửa UFW
```bash
# 1. Bắt buộc mở cổng SSH (để không bị mất quyền điều khiển VPS)
sudo ufw allow 22/tcp

# 2. Mở cổng dịch vụ Web 8085
sudo ufw allow 8085/tcp

# 3. Kích hoạt UFW (Nhấn 'y' rồi Enter nếu terminal hỏi xác nhận)
sudo ufw enable

# 4. Kiểm tra trạng thái chi tiết của tường lửa
sudo ufw status verbose
```
*Kết quả hiển thị:*
```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
8085/tcp                   ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
8085/tcp (v6)              ALLOW IN    Anywhere (v6)
```

📸 **CHỤP ẢNH MINH CHỨNG SỐ 3:**
Chụp lại màn hình kết quả lệnh `sudo ufw status verbose` -> Lưu file là: **`03-ufw.png`**.

---

## BƯỚC 6: KIỂM TRA TRUY CẬP WEBSITE & LỊCH SỬ GIT

### 6.1. Kiểm tra bằng lệnh Curl trên VPS
```bash
curl -I http://221.121.4.67:8085
```
*Kết quả trả về:* `HTTP/1.1 200 OK`.

### 6.2. Kiểm tra trên trình duyệt Web cá nhân
Mở trình duyệt (Chrome/Edge) trên máy tính cá nhân và truy cập:
```text
http://221.121.4.67:8085
```
Trang web hiển thị giao diện thông tin sinh viên *Lê Phương Linh - HNK24CNTT1*.

📸 **CHỤP ẢNH MINH CHỨNG SỐ 4:**
Chụp lại toàn bộ cửa sổ trình duyệt (nhìn thấy rõ thanh địa chỉ `http://221.121.4.67:8085` và nội dung trang web) -> Lưu file là: **`04-website.png`**.

---

### 6.3. Kiểm tra lịch sử Git log trên VPS
Trên Terminal của VPS, di chuyển vào thư mục dự án và xem lịch sử commit:
```bash
cd /var/www/devops-hackathon-de001-lephuonglinh
git log --oneline
```

📸 **CHỤP ẢNH MINH CHỨNG SỐ 5:**
Chụp lại màn hình hiển thị danh sách các commit -> Lưu file là: **`05-git-log.png`**.

---

## BƯỚC 7: MÔ PHỎNG QUY TRÌNH CẬP NHẬT WEBSITE (GIT PULL WORKFLOW)

Đây là bài kiểm tra khả năng cập nhật website không gián đoạn dịch vụ:

### 7.1. Chỉnh sửa nội dung trên máy tính (IntelliJ IDEA)
1. Mở file `src/index.html` trong IntelliJ IDEA.
2. Tìm đến phần cuối trang và thêm dòng cập nhật lần 2:
```html
        <div class="footer">
            <p><strong>Cập nhật lần 2</strong> - Hoàn thành triển khai Nginx & UFW thành công!</p>
        </div>
```
3. Lưu file lại.

### 7.2. Commit và Push thay đổi lên GitHub
Mở Terminal trên IntelliJ IDEA và chạy:
```bash
git add src/index.html
git commit -m "Cập nhật lần 2"
git push origin main
```

### 7.3. Kéo mã nguồn mới về VPS bằng `git pull`
Quay lại Terminal của VPS (hoặc cửa sổ SSH), chạy:
```bash
cd /var/www/devops-hackathon-de001-lephuonglinh
git pull
```
*Kết quả:* Git báo tải về thành công: `Fast-forward ... src/index.html | ...`.
> 💡 **Điểm mấu chốt:** Nginx đọc trực tiếp file tĩnh từ thư mục `src/`, vì vậy bạn **không cần reload hay restart Nginx**, trang web sẽ tự động hiển thị nội dung mới ngay lập tức!

### 7.4. Kiểm tra trên trình duyệt
Quay lại trình duyệt web và nhấn **F5** (Refresh). Dòng chữ *"Cập nhật lần 2"* xuất hiện trên trang web.

📸 **CHỤP ẢNH MINH CHỨNG SỐ 6:**
Chụp lại màn hình trình duyệt có hiển thị dòng chữ *"Cập nhật lần 2"* -> Lưu file là: **`06-update.png`**.

---

## BƯỚC 8: ĐƯA ẢNH MINH CHỨNG LÊN GITHUB & HOÀN TẤT BÀI THI

### 8.1. Đưa 6 ảnh minh chứng vào dự án trên IntelliJ
1. Copy 6 file ảnh bạn vừa chụp (`01-user.png` đến `06-update.png`) vào thư mục `screenshots/` của dự án trên máy tính.
2. Kiểm tra cây thư mục trong IntelliJ IDEA đảm bảo có đủ 6 ảnh:
```text
screenshots/
├── 01-user.png
├── 02-nginx.png
├── 03-ufw.png
├── 04-website.png
├── 05-git-log.png
└── 06-update.png
```

### 8.2. Push toàn bộ ảnh lên GitHub
Chạy lệnh trên Terminal của IntelliJ:
```bash
git add screenshots/
git commit -m "Thêm ảnh minh chứng"
git push origin main
```

---

## BẢNG TỔNG HỢP LỆNH KIỂM TRA NHANH TRƯỚC KHI KẾT THÚC

| Mục tiêu kiểm tra | Câu lệnh Terminal | Kết quả mong đợi |
| :--- | :--- | :--- |
| **Tài khoản đang dùng** | `whoami` | `lplinh-HNK24CNTT1` |
| **Quyền sở hữu thư mục web** | `ls -ld /var/www/devops-hackathon-de001-lephuonglinh` | `lplinh-HNK24CNTT1 lplinh-HNK24CNTT1` |
| **Cú pháp Nginx** | `sudo nginx -t` | `syntax is ok` & `test is successful` |
| **Trạng thái Nginx** | `sudo systemctl is-active nginx` | `active` |
| **Quy tắc Tường lửa** | `sudo ufw status numbered` | Cổng 22 và 8085 đều `ALLOW IN` |
| **Phản hồi HTTP Server** | `curl -I http://221.121.4.67:8085` | `HTTP/1.1 200 OK` |
| **File trên GitHub** | Truy cập link repo trên trình duyệt | Đủ `.gitignore`, `src/`, `nginx/`, `screenshots/` (6 ảnh) |
