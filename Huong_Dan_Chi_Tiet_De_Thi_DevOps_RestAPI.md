# HƯỚNG DẪN CHI TIẾT THỰC HIỆN ĐỀ THI THỰC HÀNH DEVOPS
## Ứng dụng: Spring Boot (Java 17, Gradle) & MySQL
> **Dự án thực hành minh họa**: `RestAPI_Session04`  
> **Mục tiêu**: Hướng dẫn chi tiết từng câu lệnh, từng bước thao tác, giải thích kỹ thuật và **kết quả đầu ra đạt được sau mỗi bước** để đạt trọn vẹn 10.0 điểm.

---

## MỤC LỤC
1. [Chuẩn bị môi trường & Thông tin giả định](#chuẩn-bị-môi-trường--thông-tin-giả-định)
2. [BÀI 1: Quản trị VPS Linux (2.0 điểm)](#bài-1-quản-trị-vps-linux-20-điểm)
   - [Yêu cầu 1.1: Kết nối & Quản lý tài khoản (0.5 điểm)](#yêu-cầu-11--kết-nối-và-quản-lý-tài-khoản-05-điểm)
   - [Yêu cầu 1.2: Quản lý nhóm & Quyền Docker (0.5 điểm)](#yêu-cầu-12--quản-lý-nhóm-và-quyền-05-điểm)
   - [Yêu cầu 1.3: Quản lý file, thư mục & Phân quyền (1.0 điểm)](#yêu-cầu-13--quản-lý-file-và-thư-mục-10-điểm)
3. [BÀI 2: Docker trên VPS (2.0 điểm)](#bài-2-docker-trên-vps-20-điểm)
   - [Yêu cầu 2.1: Hoàn thiện Dockerfile cho Spring Boot (0.75 điểm)](#yêu-cầu-21--hoàn-thiện-dockerfile-075-điểm)
   - [Yêu cầu 2.2: Hoàn thiện docker-compose.yml (0.75 điểm)](#yêu-cầu-22--hoàn-thiện-docker-compose-075-điểm)
   - [Yêu cầu 2.3: Khởi chạy và kiểm tra Docker (0.5 điểm)](#yêu-cầu-23--kiểm-tra-docker-05-điểm)
4. [BÀI 3: Git & GitHub (1.5 điểm)](#bài-3-git-và-github-15-điểm)
   - [Yêu cầu 3.1: Tạo Git Repo Local & Remote GitHub (0.5 điểm)](#yêu-cầu-31--tạo-repository-05-điểm)
   - [Yêu cầu 3.2: Tạo nhánh tính năng & Commit (0.5 điểm)](#yêu-cầu-32--commit-và-phát-triển-chức-năng-05-điểm)
   - [Yêu cầu 3.3: Tạo kịch bản & Xử lý Merge Conflict (0.5 điểm)](#yêu-cầu-33--xử-lý-conflict-05-điểm)
5. [BÀI 4: Phát triển Chức năng & Triển khai CI/CD (4.5 điểm)](#bài-4-phát-triển-chức-năng-và-triển-khai-cicd-45-điểm)
   - [Yêu cầu 4.1: Phát triển API mới (GET /api/employees/count) (1.0 điểm)](#yêu-cầu-41--phát-triển-api-mới-10-điểm)
   - [Yêu cầu 4.2: Cấu hình Secrets & Workflow GitHub Actions (1.5 điểm)](#yêu-cầu-42--xây-dựng-github-actions-cicd-15-điểm)
   - [Yêu cầu 4.3: Tự động triển khai & Kiểm chứng thực tế (1.5 điểm)](#yêu-cầu-43--triển-khai-và-kiểm-chứng-15-điểm)
6. [Bảng tổng hợp Troubleshooting & Lỗi thường gặp](#bảng-tổng-hợp-troubleshooting--lỗi-thường-gặp)

---

## Chuẩn bị môi trường & Thông tin giả định

Trong suốt bài hướng dẫn này, các tham số sau được quy ước làm ví dụ (hãy thay thế bằng thông tin thực tế của bạn khi làm bài):
- **IP VPS**: `103.179.189.150` (hoặc IP VPS do trường cấp)
- **User quản trị ban đầu**: `root` hoặc `ubuntu`
- **User sinh viên tạo mới**: `devops_student`
- **Mã sinh viên**: `B21DCCN001`
- **Tên GitHub Repository**: `devops-exam-B21DCCN001`
- **Tài khoản Docker Hub**: `mydevopsuser`
- **Tên Docker Image**: `mydevopsuser/restapi-session04:latest`
- **Cổng Backend ứng dụng**: `8080`
- **Cổng MySQL trên host (nếu public)**: `3306`

---

## BÀI 1: Quản trị VPS Linux (2.0 điểm)

> **Quy định bắt buộc**: Tuyệt đối không dùng tài khoản `root` để thao tác git, build source code hay deploy thường ngày.

### Yêu cầu 1.1 — Kết nối và quản lý tài khoản (0.5 điểm)

#### Bước 1.1.1: Đăng nhập vào VPS qua SSH
Mở Terminal trên máy tính (PowerShell, Git Bash hoặc Terminal Linux/macOS) và thực hiện kết nối:
```bash
ssh root@103.179.189.150
# Hoặc nếu dùng SSH Key có sẵn:
# ssh -i ~/.ssh/my_vps_key root@103.179.189.150
```

#### Bước 1.1.2: Kiểm tra tài khoản hiện tại
Sau khi vào VPS, chạy lệnh kiểm tra danh tính:
```bash
whoami
id
```
- **Kết quả đạt được**:
  ```text
  root
  uid=0(root) gid=0(root) groups=0(root)
  ```
  *(Thể hiện bạn đang đứng ở quyền root ban đầu)*.

#### Bước 1.1.3: Tạo tài khoản Linux mới `devops_student`
Sử dụng lệnh `adduser` (lệnh này tự động tạo home directory và yêu cầu thiết lập mật khẩu):
```bash
adduser devops_student
```
- Hệ thống sẽ yêu cầu nhập mật khẩu (ví dụ: `Password123@`) và xác nhận lại mật khẩu.
- Các thông tin `Full Name`, `Room Number`,... nhấn `Enter` để bỏ qua, gõ `Y` để xác nhận.

Cấp quyền thực thi lệnh quản trị (`sudo`) cho tài khoản mới:
```bash
usermod -aG sudo devops_student
```

#### Bước 1.1.4: Chuyển sang tài khoản mới và kiểm tra
```bash
su - devops_student
whoami
pwd
```
- **Kết quả đạt được**:
  ```text
  devops_student
  /home/devops_student
  ```
  Prompt dòng lệnh chuyển thành dạng `devops_student@vps:~$`.

---

### Yêu cầu 1.2 — Quản lý nhóm và quyền (0.5 điểm)

#### Bước 1.2.1: Kiểm tra thông tin nhóm hiện tại
```bash
id
```
- **Kết quả đạt được**:
  ```text
  uid=1001(devops_student) gid=1001(devops_student) groups=1001(devops_student),27(sudo)
  ```

#### Bước 1.2.2: Kiểm tra nhóm `docker` có tồn tại trên VPS hay không
```bash
getent group docker
# hoặc:
grep docker /etc/group
```
- **Kết quả đạt được**:
  ```text
  docker:x:998:
  ```
  *(Nếu chưa có nhóm docker, chạy lệnh: `sudo groupadd docker`)*.

#### Bước 1.2.3: Bổ sung tài khoản vào nhóm `docker`
```bash
sudo usermod -aG docker devops_student
```
> **LƯU Ý CỰC KỲ QUAN TRỌNG**: Sau khi thêm user vào group, quyền chưa có hiệu lực ngay trong phiên làm việc hiện tại. Bạn phải áp dụng ngay bằng lệnh:
```bash
newgrp docker
```
Kiểm tra lại bằng `id`:
```bash
id
```
- **Kết quả đạt được**: Xuất hiện `docker` trong danh sách `groups`:
  ```text
  uid=1001(devops_student) gid=1001(devops_student) groups=1001(devops_student),27(sudo),998(docker)
  ```

#### Bước 1.2.4: Kiểm tra quyền Docker không cần `sudo`
```bash
docker ps
```
- **Kết quả đạt được**: Lệnh chạy trơn tru, không báo lỗi `permission denied while trying to connect to the Docker daemon socket`.

#### Bước 1.2.5: Trả lời câu hỏi phỏng vấn/lý thuyết đề thi:
> **Câu hỏi**: *"Vì sao thành viên nhóm docker có quyền tương đương quyền quản trị rất cao (root) trên VPS?"*  
> **Trả lời**:  
> Docker Daemon (`dockerd`) chạy mặc định dưới quyền `root` và giao tiếp qua Unix socket `/var/run/docker.sock`. Mặc định Unix socket này thuộc sở hữu của nhóm `docker`. Khi một user được thêm vào nhóm `docker`, user đó có thể gửi lệnh trực tiếp tới Docker Daemon mà không cần `sudo`. Từ đó, user hoàn toàn có thể khởi tạo một container mount toàn bộ thư mục gốc của máy chủ VPS (`docker run -v /:/host -it ubuntu bash`) và tùy ý chỉnh sửa file `/etc/shadow`, tạo user root mới hoặc thay đổi bất kỳ file hệ thống nào của VPS. Do đó, cấp quyền nhóm `docker` tương đương với cấp quyền root.

---

### Yêu cầu 1.3 — Quản lý file và thư mục (1.0 điểm)

Đảm bảo bạn đang đứng ở thư mục gốc của tài khoản (`/home/devops_student`):
```bash
cd ~
pwd
```

#### Bước 1.3.1: Tạo thư mục `devops-exam`
```bash
mkdir -p devops-exam
cd devops-exam
```

#### Bước 1.3.2: Tạo file `info.txt`
Sử dụng `nano` hoặc lệnh chuyển hướng dòng `cat`:
```bash
cat << 'EOF' > info.txt
Họ và tên: Nguyễn Văn A
Mã sinh viên: B21DCCN001
GitHub Repo: https://github.com/myusername/devops-exam-B21DCCN001
EOF
```

#### Bước 1.3.3: Xem nội dung và thông tin file
```bash
cat info.txt
ls -l info.txt
```
- **Kết quả đạt được**:
  ```text
  Họ và tên: Nguyễn Văn A
  Mã sinh viên: B21DCCN001
  GitHub Repo: https://github.com/myusername/devops-exam-B21DCCN001
  -rw-rw-r-- 1 devops_student devops_student 112 Oct 14 14:00 info.txt
  ```

#### Bước 1.3.4: Thiết lập và kiểm tra quyền truy cập (`chmod`, `chown`)
Giả sử yêu cầu bảo mật: Chỉ cho phép chủ sở hữu đọc/ghi (`rw-`), nhóm và người ngoài không có quyền (`---`) -> Quyền `600`:
```bash
chmod 600 info.txt
chown devops_student:devops_student info.txt
ls -la info.txt
```
- **Kết quả đạt được**:
  ```text
  -rw------- 1 devops_student devops_student 112 Oct 14 14:00 info.txt
  ```
> **Minh chứng Bài 1**: Chụp ảnh màn hình hiển thị toàn bộ terminal có dấu nhắc lệnh `devops_student@...`, kết quả của `whoami`, `id`, `ls -la info.txt` và nội dung file `info.txt`.

---

## BÀI 2: Docker trên VPS (2.0 điểm)

Ứng dụng Spring Boot của dự án [RestAPI_Session04](file:///d:/DevOps%20-%20RA/RestAPI_Session04) sử dụng **Java 17** và trình build **Gradle**. Ta sẽ hoàn thiện [Dockerfile](file:///d:/DevOps%20-%20RA/RestAPI_Session04/Dockerfile) và [docker-compose.yml](file:///d:/DevOps%20-%20RA/RestAPI_Session04/docker-compose.yml) tối ưu, chuẩn quy chuẩn DevOps.

### Yêu cầu 2.1 — Hoàn thiện Dockerfile (0.75 điểm)

Có 2 mô hình viết Dockerfile:
1. **Mô hình CI/CD Build trước (Khuyên dùng theo luồng đề thi)**: Pipeline build ra file `.jar`, Dockerfile chỉ cần đóng gói Runtime (nhẹ, build cực nhanh ~ 10 giây).
2. **Mô hình Multi-stage Build**: Docker tự build mã nguồn bằng Gradle rồi đóng gói.

Dưới đây là Dockerfile chuẩn mực tối ưu cho `RestAPI_Session04` (dùng base image **Eclipse Temurin 17 JRE** nhẹ và ổn định):

Tạo file `Dockerfile` trong thư mục gốc của project:
```dockerfile
# Sử dụng base image JRE 17 Alpine gọn nhẹ, bảo mật
FROM eclipse-temurin:17-jre-alpine

# Thiết lập thư mục làm việc trong container
WORKDIR /app

# Khai báo argument cho file JAR (mặc định của Gradle nằm trong build/libs/)
ARG JAR_FILE=build/libs/*.jar

# Copy file jar sau khi đã build vào image
COPY ${JAR_FILE} app.jar

# Mở cổng 8080 của ứng dụng Spring Boot
EXPOSE 8080

# Cấu hình biến môi trường JVM tối ưu bộ nhớ container
ENV JAVA_OPTS="-Xms256m -Xmx512m"

# Lệnh khởi chạy ứng dụng Spring Boot
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

- **Giải thích kỹ thuật**:
  - `eclipse-temurin:17-jre-alpine`: Phù hợp hoàn hảo với `JavaLanguageVersion.of(17)` trong `build.gradle`.
  - `build/libs/*.jar`: Đường dẫn đầu ra khi chạy lệnh `./gradlew bootJar`.
  - `ENTRYPOINT`: Cho phép truyền biến môi trường `JAVA_OPTS` linh hoạt khi chạy container.

---

### Yêu cầu 2.2 — Hoàn thiện Docker Compose (0.75 điểm)

Khi chạy ứng dụng Spring Boot cùng MySQL, file `docker-compose.yml` phải giải quyết 3 vấn đề:
1. **Network**: Backend và MySQL phải cùng chung một mạng để backend gọi MySQL qua tên hostname (ví dụ: `mysql-db`).
2. **Persistent Volume**: Dữ liệu MySQL không bị mất khi hạ container.
3. **Environment Override**: Override lại thông số kết nối database trong `application.properties` mà không cần sửa code.

Tạo file `docker-compose.yml`:
```yaml
services:
  # ------------------------------------
  # 1. DỊCH VỤ DATABASE: MYSQL 8.0
  # ------------------------------------
  mysql-db:
    image: mysql:8.0
    container_name: restapi_mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 1234$
      MYSQL_DATABASE: db_cntt3_api_ss4
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p1234$"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ------------------------------------
  # 2. DỊCH VỤ BACKEND: SPRING BOOT
  # ------------------------------------
  backend:
    build:
      context: .
      dockerfile: Dockerfile
    image: restapi-session04:latest
    container_name: restapi_backend
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      # Override cấu hình datasource của Spring Boot trỏ về service 'mysql-db'
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql-db:3306/db_cntt3_api_ss4?createDatabaseIfNotExist=true&useSSL=false&allowPublicKeyRetrieval=true
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: 1234$
      SPRING_JPA_HIBERNATE_DDL_AUTO: update
    depends_on:
      mysql-db:
        condition: service_healthy
    networks:
      - app-network

# ------------------------------------
# KHAI BÁO VOLUMES & NETWORKS
# ------------------------------------
volumes:
  mysql_data:
    driver: local

networks:
  app-network:
    driver: bridge
```

- **Điểm sáng tạo điểm cộng trong bài thi**:
  - Dùng `healthcheck` trên MySQL và `condition: service_healthy` trên backend: Tránh tình trạng Spring Boot khởi động trước khi MySQL sẵn sàng nhận kết nối, gây lỗi `Communications link failure`.
  - Override `SPRING_DATASOURCE_URL` trỏ vào `mysql-db:3306` (tên service trong compose) thay vì `localhost`.

---

### Yêu cầu 2.3 — Kiểm tra Docker (0.5 điểm)

#### Bước 2.3.1: Build file JAR và khởi chạy Docker Compose
Trước khi chạy compose (nếu test tại chỗ):
```bash
# Cấp quyền thực thi và build JAR (bỏ qua test để build nhanh)
chmod +x ./gradlew
./gradlew bootJar -x test

# Khởi động dịch vụ ở chế độ chạy ngầm (detached mode)
docker compose up -d --build
```

#### Bước 2.3.2: Kiểm tra trạng thái các container
```bash
docker compose ps
# Hoặc:
docker ps
```
- **Kết quả đạt được**:
  ```text
  CONTAINER ID   IMAGE                    COMMAND                  STATUS                    PORTS
  a1b2c3d4e5f6   restapi-session04:latest "sh -c 'java $JAVA_O…"   Up 45 seconds             0.0.0.0:8080->8080/tcp
  f6e5d4c3b2a1   mysql:8.0                "docker-entrypoint.s…"   Up About a minute (healthy) 0.0.0.0:3306->3306/tcp
  ```
  *(Cả 2 container đều có STATUS là Up, mysql-db ở trạng thái healthy)*.

#### Bước 2.3.3: Xem danh sách Image
```bash
docker images
```
- **Kết quả đạt được**: Thấy image `restapi-session04:latest` và `mysql:8.0`.

#### Bước 2.3.4: Xem Logs của Backend để xác nhận kết nối Database thành công
```bash
docker logs restapi_backend -f
# hoặc:
docker compose logs -f backend
```
- **Kết quả đạt được**:
  Dòng log cuối cùng thể hiện:
  ```text
  Started RestApiSession04Application in 4.123 seconds (process running for 4.891)
  Tomcat started on port 8080 (http) with context path '/'
  ```
  Không xuất hiện ngoại lệ `SQLException` hay crash container. Nhấn `Ctrl + C` để thoát xem log.

---

## BÀI 3: Git & GitHub (1.5 điểm)

### Yêu cầu 3.1 — Tạo Repository (0.5 điểm)

#### Bước 3.1.1: Khởi tạo Git cục bộ trong thư mục source code
Mở terminal tại thư mục dự án `RestAPI_Session04`:
```bash
git init
git checkout -b main
```

#### Bước 3.1.2: Cấu hình `.gitignore` và Commit ban đầu
Đảm bảo các file rác của build không bị push:
```bash
# Kiểm tra file .gitignore đã có .gradle, /build, .idea,...
git status
git add .
git commit -m "feat: initial spring boot project RestAPI_Session04"
```

#### Bước 3.1.3: Tạo GitHub Repo và kết nối Remote
1. Truy cập [GitHub](https://github.com/new), tạo repository mới:
   - **Tên Repo**: `devops-exam-B21DCCN001` (đặt đúng theo mã SV của bạn).
   - **Chế độ**: Public (hoặc Private theo yêu cầu giám thị).
2. Kết nối và push lên GitHub:
```bash
git remote add origin https://github.com/myusername/devops-exam-B21DCCN001.git
git branch -M main
git push -u origin main
```
- **Kết quả đạt được**: Source code đã xuất hiện đầy đủ trên GitHub tại nhánh `main`.

---

### Yêu cầu 3.2 — Commit và phát triển chức năng (0.5 điểm)

#### Bước 3.2.1: Tạo và chuyển sang nhánh `feature/employee-count`
```bash
git checkout -b feature/employee-count
```

#### Bước 3.2.2: Thực hiện chỉnh sửa mã nguồn
(Ví dụ: Bạn tạo file hoặc chỉnh sửa ghi chú trong `README.md` hoặc code trực tiếp chức năng đếm nhân viên):
```bash
echo "Tính năng đếm nhân viên đang được phát triển" >> README.md
git add README.md
git commit -m "feat(employee): add employee count feature documentation"
git push -u origin feature/employee-count
```
- **Kết quả đạt được**: Nhánh `feature/employee-count` được đẩy thành công lên GitHub.

---

### Yêu cầu 3.3 — Xử lý Conflict (0.5 điểm)

Để ăn trọn 0.5 điểm phần này, ta tạo một kịch bản **Merge Conflict có chủ đích** giữa nhánh `main` và `feature/employee-count`:

#### Bước 3.3.1: Tạo thay đổi trên nhánh `main`
Chuyển về `main` và sửa cùng một dòng trong file `README.md`:
```bash
git checkout main
echo "Phiên bản hệ thống: Version 1.0.0 Production" >> README.md
git add README.md
git commit -m "chore: update system version on main"
```

#### Bước 3.3.2: Tạo thay đổi xung đột trên nhánh `feature/employee-count`
Chuyển sang nhánh feature và sửa cùng vị trí:
```bash
git checkout feature/employee-count
echo "Phiên bản hệ thống: Version 1.1.0 Beta Employee Counter" >> README.md
git add README.md
git commit -m "chore: update system version on feature branch"
```

#### Bước 3.3.3: Thực hiện Merge vào `main` và đón nhận Conflict
```bash
git checkout main
git merge feature/employee-count
```
- **Kết quả đạt được (Xuất hiện Conflict)**:
  ```text
  Auto-merging README.md
  CONFLICT (content): Merge conflict in README.md
  Automatic merge failed; fix conflicts and then commit the result.
  ```

#### Bước 3.3.4: Giải quyết Conflict
Mở file `README.md`, Git sẽ đánh dấu xung đột dạng:
```markdown
<<<<<<< HEAD
Phiên bản hệ thống: Version 1.0.0 Production
=======
Phiên bản hệ thống: Version 1.1.0 Beta Employee Counter
>>>>>>> feature/employee-count
```
Chỉnh sửa lại file để giữ nội dung mong muốn (ví dụ kết hợp cả hai hoặc giữ bản hoàn thiện):
```markdown
Phiên bản hệ thống: Version 1.1.0 Production (Included Employee Counter)
```

Lưu file và hoàn tất merge:
```bash
git add README.md
git commit -m "merge: resolve merge conflict between main and feature/employee-count"
git push origin main
```

#### Bước 3.3.5: Kiểm tra lịch sử đồ thị Git
```bash
git log --oneline --graph --all
```
- **Kết quả đạt được**:
  ```text
  *   3a1b2c4 (HEAD -> main, origin/main) merge: resolve merge conflict between main and feature/employee-count
  |\  
  | * 5e6f7a8 (origin/feature/employee-count, feature/employee-count) chore: update system version on feature branch
  * | 9c8b7a6 chore: update system version on main
  |/  
  * 1f2e3d4 feat: initial spring boot project RestAPI_Session04
  ```
> **Minh chứng Bài 3**: Chụp ảnh màn hình terminal hiển thị kết quả lệnh `git log --oneline --graph --all` có đầy đủ các nhánh rẽ và điểm hợp nhất (merge commit).

---

## BÀI 4: Phát triển Chức năng & Triển khai CI/CD (4.5 điểm)

### Yêu cầu 4.1 — Phát triển API mới (1.0 điểm)
**Đề bài yêu cầu**:
- Method: `GET`
- Endpoint: `/api/employees/count`
- Dữ liệu trả về: Số lượng nhân viên hiện có (kiểu số nguyên / JSON, ví dụ: nếu có 5 nhân viên thì trả về số `5`).

Dưới đây là mã nguồn chuẩn theo kiến trúc Spring Boot (Entity $\rightarrow$ Repository $\rightarrow$ Service $\rightarrow$ Controller):

#### 1. Entity `Employee.java`
Tạo tại `src/main/java/ra/demo/model/entity/Employee.java`:
```java
package ra.demo.model.entity;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "employees")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String fullName;

    @Column(unique = true, nullable = false, length = 100)
    private String email;

    private String department;
}
```

#### 2. Repository `EmployeeRepository.java`
Tạo tại `src/main/java/ra/demo/repository/EmployeeRepository.java`:
```java
package ra.demo.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import ra.demo.model.entity.Employee;

@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    // JpaRepository đã có sẵn phương thức long count();
}
```

#### 3. Service Interface & Impl
Tạo interface `src/main/java/ra/demo/service/EmployeeService.java`:
```java
package ra.demo.service;

public interface EmployeeService {
    long countEmployees();
}
```

Tạo class triển khai `src/main/java/ra/demo/service/impl/EmployeeServiceImpl.java`:
```java
package ra.demo.service.impl;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import ra.demo.repository.EmployeeRepository;
import ra.demo.service.EmployeeService;

@Service
@RequiredArgsConstructor
public class EmployeeServiceImpl implements EmployeeService {
    private final EmployeeRepository employeeRepository;

    @Override
    public long countEmployees() {
        return employeeRepository.count();
    }
}
```

#### 4. Controller `EmployeeController.java`
Tạo tại `src/main/java/ra/demo/controller/EmployeeController.java`:
```java
package ra.demo.controller;

import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import ra.demo.service.EmployeeService;

@RestController
@RequestMapping("/api/employees")
@RequiredArgsConstructor
public class EmployeeController {
    private final EmployeeService employeeService;

    @GetMapping("/count")
    public ResponseEntity<Long> countEmployees() {
        long total = employeeService.countEmployees();
        return ResponseEntity.ok(total);
    }
}
```

> *(Ghi chú bổ sung: Nếu trong đề thi cho phép dùng entity `Product` có sẵn trong RestAPI_Session04, bạn chỉ cần thêm endpoint `@GetMapping("/count")` trong `ProductController` trả về `productRepository.count()`, nguyên lý hoàn toàn tương đương)*.

---

### Yêu cầu 4.2 — Xây dựng GitHub Actions CI/CD (1.5 điểm)

#### Bước 4.2.1: Chuẩn bị SSH Key để GitHub Actions kết nối vào VPS
Trên VPS hoặc máy tính của bạn, tạo một cặp khóa SSH dành riêng cho CI/CD:
```bash
ssh-keygen -t rsa -b 4096 -C "github-actions-deploy" -f ~/.ssh/id_rsa_deploy -N ""
```
- Đưa Public Key vào danh sách cho phép đăng nhập của user `devops_student` trên VPS:
```bash
cat ~/.ssh/id_rsa_deploy.pub >> /home/devops_student/.ssh/authorized_keys
chmod 700 /home/devops_student/.ssh
chmod 600 /home/devops_student/.ssh/authorized_keys
chown -R devops_student:devops_student /home/devops_student/.ssh
```
- Lấy Private Key để cấu hình GitHub Secrets:
```bash
cat ~/.ssh/id_rsa_deploy
```
*(Copy toàn bộ nội dung từ `-----BEGIN OPENSSH PRIVATE KEY-----` đến `-----END OPENSSH PRIVATE KEY-----`)*.

#### Bước 4.2.2: Cấu hình GitHub Actions Secrets
Vào GitHub Repository: **Settings** $\rightarrow$ **Secrets and variables** $\rightarrow$ **Actions** $\rightarrow$ **New repository secret**. Thêm 5 secrets sau:

| Tên Secret | Giá trị |
| :--- | :--- |
| `DOCKER_USERNAME` | Username đăng nhập Docker Hub (ví dụ: `mydevopsuser`) |
| `DOCKER_PASSWORD` | Docker Hub Access Token (tạo tại Docker Hub > Account Settings > Security) |
| `VPS_HOST` | Địa chỉ IP của VPS (ví dụ: `103.179.189.150`) |
| `VPS_USER` | `devops_student` |
| `VPS_SSH_KEY` | Nội dung Private Key đã copy ở bước trên |
| `VPS_PORT` | `22` (cổng SSH) |

#### Bước 4.2.3: Viết Workflow `.github/workflows/deploy.yml`
Tạo file `.github/workflows/deploy.yml` trong mã nguồn dự án:

```yaml
name: CI/CD Pipeline - RestAPI Spring Boot

on:
  push:
    branches: [ "main" ]

jobs:
  # ==========================================
  # GIAI ĐOẠN 1: BUILD & TEST (CI)
  # ==========================================
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: 1. Checkout mã nguồn
        uses: actions/checkout@v4

      - name: 2. Thiết lập JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: gradle

      - name: 3. Cấp quyền thực thi cho Gradle Wrapper
        run: chmod +x ./gradlew

      - name: 4. Build ứng dụng Spring Boot (JAR)
        # Bỏ qua test nếu chưa có DB trên GitHub runner để tránh fail pipeline
        run: ./gradlew bootJar -x test

      - name: 5. Đăng nhập Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: 6. Build và Push Docker Image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/restapi-session04:latest

  # ==========================================
  # GIAI ĐOẠN 2: DEPLOY LÊN VPS (CD)
  # ==========================================
  deploy-to-vps:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
      - name: 1. Checkout repository để lấy file docker-compose.yml
        uses: actions/checkout@v4

      - name: 2. Copy docker-compose.yml sang VPS
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          port: ${{ secrets.VPS_PORT || 22 }}
          source: "docker-compose.yml"
          target: "/home/${{ secrets.VPS_USER }}/devops-exam/"

      - name: 3. SSH vào VPS và triển khai phiên bản mới
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          port: ${{ secrets.VPS_PORT || 22 }}
          script: |
            cd /home/${{ secrets.VPS_USER }}/devops-exam/
            
            # Cập nhật image mới nhất từ Docker Hub trong file docker-compose nếu cần
            # Pull image mới về VPS
            docker pull ${{ secrets.DOCKER_USERNAME }}/restapi-session04:latest
            
            # Khởi động lại container backend với image mới
            docker compose down backend || true
            docker compose up -d
            
            # Dọn dẹp các images không còn sử dụng (dangling images)
            docker image prune -f
            
            # Hiển thị trạng thái các container sau khi deploy
            docker ps
```

---

### Yêu cầu 4.3 — Triển khai và kiểm chứng (1.5 điểm)

#### Bước 4.3.1: Commit và Push mã nguồn kích hoạt Pipeline
```bash
git add .
git commit -m "feat: implement /api/employees/count and setup complete CI/CD workflow"
git push origin main
```

#### Bước 4.3.2: Kiểm tra trạng thái trên GitHub Actions
1. Truy cập tab **Actions** trên GitHub repository.
2. Quan sát workflow `CI/CD Pipeline - RestAPI Spring Boot` đang chạy.
3. Khi cả 2 job `build-and-test` và `deploy-to-vps` đều hiện tích xanh ✅:
   - File JAR đã build thành công.
   - Image đã được đẩy lên Docker Hub: `mydevopsuser/restapi-session04:latest`.
   - VPS đã kéo image và khởi động lại container backend.

#### Bước 4.3.3: Kiểm chứng trực tiếp trên VPS
Đăng nhập vào VPS với user `devops_student`:
```bash
docker ps
docker logs restapi_backend --tail 50
```
- **Kết quả đạt được**: Container đang chạy (`Up`), log hiển thị Spring Boot đã khởi động thành công trên port 8080.

#### Bước 4.3.4: Gọi API kiểm chứng tính năng bằng `curl`
Chạy lệnh `curl` từ máy tính cá nhân hoặc ngay trên terminal VPS:
```bash
curl -X GET http://103.179.189.150:8080/api/employees/count
```
- **Kết quả mong đợi trả về**:
  ```text
  0
  ```
  *(Số 0 biểu thị cơ sở dữ liệu mới khởi tạo chưa có nhân viên nào)*.

Thêm thử 1 bản ghi vào MySQL trên VPS để kiểm tra:
```bash
docker exec -it restapi_mysql mysql -uroot -p'1234$' -e "USE db_cntt3_api_ss4; INSERT INTO employees (full_name, email, department) VALUES ('Nguyen Van A', 'a@gmail.com', 'IT');"
```

Gọi lại lệnh `curl`:
```bash
curl -X GET http://103.179.189.150:8080/api/employees/count
```
- **Kết quả đạt được**:
  ```text
  1
  ```
> **Minh chứng Bài 4**: Chụp ảnh tab GitHub Actions xanh 100% kèm ảnh chụp terminal chạy lệnh `curl http://...:8080/api/employees/count` trả về kết quả số lượng nhân viên chính xác.

---

## Bảng tổng hợp Troubleshooting & Lỗi thường gặp

| Lỗi gặp phải | Nguyên nhân | Cách khắc phục triệt để |
| :--- | :--- | :--- |
| `permission denied while trying to connect to Docker daemon` | User chưa có quyền trong nhóm `docker` hoặc chưa reload session. | Chạy `sudo usermod -aG docker devops_student` sau đó gõ lệnh `newgrp docker`. |
| `Host key verification failed` trong GitHub Actions | SSH Action chưa tin cậy host key của VPS. | Thêm param `fingerprint` hoặc action `appleboy/ssh-action` tự động xử lý khi khai báo đủ host, user và key. |
| `Communications link failure` khi Spring Boot khởi động | Container MySQL khởi động chậm hơn Spring Boot. | Dùng `healthcheck` trên MySQL và `condition: service_healthy` ở `depends_on` trong `docker-compose.yml`. |
| `Permission denied: ./gradlew` trong GitHub Actions | File `gradlew` mất cờ thực thi khi push lên Git từ Windows. | Thêm step `run: chmod +x ./gradlew` trước khi chạy `./gradlew bootJar`. |
| `Load key "...": error in libcrypto` | SSH Private Key copy vào GitHub Secret bị thiếu dòng header/footer hoặc bị ngắt dòng sai. | Copy chính xác từ `-----BEGIN OPENSSH PRIVATE KEY-----` đến `-----END OPENSSH PRIVATE KEY-----` kèm đầy đủ ký tự xuống dòng. |
| MySQL lỗi `Access denied for user 'root'@'...'` | Mật khẩu khai báo trong biến môi trường không khớp giữa MySQL và Backend. | Đảm bảo `MYSQL_ROOT_PASSWORD` trong mysql-db bằng chính xác `SPRING_DATASOURCE_PASSWORD` trong backend. |
