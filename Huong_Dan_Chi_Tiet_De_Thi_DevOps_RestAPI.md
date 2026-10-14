# HƯỚNG DẪN CHI TIẾT THỰC HIỆN ĐỀ THI THỰC HÀNH DEVOPS
## Ứng dụng: Spring Boot (Java 17, Gradle) & MySQL
> **Dự án thực hành minh họa**: `RestAPI_Session04`  
> **Địa chỉ IP VPS thực tế**: `221.121.4.132`  
> **GitHub Username**: `phuonglinhle268`  
> **Tên GitHub Repository**: `PTIT_CNTT1_IT209_Demo_Final`  
> **Docker Hub Username**: `phuonglinhle`  
> **Linux Username sinh viên**: `lplinh_k24cntt1`  
> **Mục tiêu**: Hướng dẫn thực hành có phân định rõ ràng vị trí thực hiện (**🖥️ VPS**, **💻 Máy Local**, **🌐 Trình duyệt**), **chỉ chú thích các vị trí cần sửa khi đổi sang dự án khác**, không thêm giải thích rườm rà.

---

## Chuẩn bị môi trường & Thông tin tham số thực tế

Các tham số sử dụng trong toàn bộ bài thi:
- **Địa chỉ IP VPS**: `221.121.4.132`
- **User quản trị ban đầu của VPS**: `root`
- **User sinh viên tạo mới**: `lplinh_k24cntt1`
- **Tài khoản GitHub**: `phuonglinhle268`
- **Tên GitHub Repository**: `PTIT_CNTT1_IT209_Demo_Final`
- **Tài khoản Docker Hub**: `phuonglinhle`
- **Tên Docker Image**: `phuonglinhle/ptit-demo-final:latest`
- **Cổng Backend ứng dụng**: `8080`
- **Cổng MySQL trên host**: `3306` (hoặc `3307`)

---

## BƯỚC 0: Khởi tạo & Cài đặt gói công cụ trên VPS mới hoàn toàn
> 🖥️ **VỊ TRÍ THỰC HIỆN**: **Trên Terminal VPS (kết nối SSH bằng quyền root)**

### 0.1 Cập nhật hệ thống & cài các tiện ích cơ bản
```bash
ssh root@221.121.4.132
apt update && apt upgrade -y
apt install -y curl wget git nano net-tools ca-certificates gnupg
```
- **Kết quả đạt được**: Các gói phần mềm được cài đặt thành công.

### 0.2 Cài đặt Docker Engine & Docker Compose chính thức
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
systemctl enable --now docker
docker --version
docker compose version
```
- **Kết quả đạt được**:
  ```text
  Docker version 27.x.x, build ...
  Docker Compose version v2.x.x
  ```

---

## BÀI 1: Quản trị VPS Linux (2.0 điểm)
> 🖥️ **VỊ TRÍ THỰC HIỆN**: **100% trên Terminal VPS (SSH: `221.121.4.132`)**  
> **Quy định bắt buộc**: Không dùng tài khoản `root` để thao tác git, build source code hay deploy thường ngày.

### Yêu cầu 1.1 — Kết nối và quản lý tài khoản (0.5 điểm)

#### Bước 1.1.1: Kiểm tra tài khoản root ban đầu
```bash
whoami
id
```
- **Kết quả đạt được**:
  ```text
  root
  uid=0(root) gid=0(root) groups=0(root)
  ```

#### Bước 1.1.2: Tạo tài khoản Linux mới `lplinh_k24cntt1`
```bash
adduser lplinh_k24cntt1
usermod -aG sudo lplinh_k24cntt1
```
- Nhập mật khẩu (ví dụ: `Password123@`), xác nhận lại mật khẩu, các thông tin phụ khác nhấn `Enter`, gõ `Y` để xác nhận.

#### Bước 1.1.3: Chuyển sang tài khoản mới và kiểm tra
```bash
su - lplinh_k24cntt1
whoami
pwd
```
- **Kết quả đạt được**:
  ```text
  lplinh_k24cntt1
  /home/lplinh_k24cntt1
  ```
  Dấu nhắc dòng lệnh chuyển thành: `lplinh_k24cntt1@Linh-VM3:~$`.

---

### Yêu cầu 1.2 — Quản lý nhóm và quyền (0.5 điểm)

#### Bước 1.2.1: Kiểm tra thông tin nhóm hiện tại
```bash
id
```
- **Kết quả đạt được**:
  ```text
  uid=1000(lplinh_k24cntt1) gid=1000(lplinh_k24cntt1) groups=1000(lplinh_k24cntt1),27(sudo)
  ```

#### Bước 1.2.2: Kiểm tra nhóm `docker`
```bash
getent group docker
```
- **Kết quả đạt được**: Hiển thị thông tin nhóm (ví dụ: `docker:x:998:` hoặc `docker:x:1001:`). Nếu chưa có thì chạy `sudo groupadd docker`.

#### Bước 1.2.3: Bổ sung tài khoản vào nhóm `docker`
```bash
sudo usermod -aG docker lplinh_k24cntt1
newgrp docker
id
```
- **Kết quả đạt được**: Trong danh sách `groups` xuất hiện `docker`.

#### Bước 1.2.4: Kiểm tra quyền Docker không cần `sudo`
```bash
docker ps
```
- **Kết quả đạt được**: Hiển thị bảng tiêu đề `CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES` và không có lỗi permission denied.

---

### Yêu cầu 1.3 — Quản lý file và thư mục (1.0 điểm)

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
```bash
cat << 'EOF' > info.txt
Họ và tên: Lê Phương Linh
Mã sinh viên: CNTT1_IT209
GitHub Repo: https://github.com/phuonglinhle268/PTIT_CNTT1_IT209_Demo_Final
EOF
```

#### Bước 1.3.3: Xem nội dung và thông tin file
```bash
cat info.txt
ls -l info.txt
```
- **Kết quả đạt được**:
  ```text
  Họ và tên: Lê Phương Linh
  Mã sinh viên: CNTT1_IT209
  GitHub Repo: https://github.com/phuonglinhle268/PTIT_CNTT1_IT209_Demo_Final
  -rw-rw-r-- 1 lplinh_k24cntt1 docker 134 Oct 14 14:00 info.txt
  ```

#### Bước 1.3.4: Thiết lập và kiểm tra quyền truy cập (`chmod`, `chown`)
```bash
chmod 600 info.txt
chown lplinh_k24cntt1:lplinh_k24cntt1 info.txt
ls -la info.txt
```
- **Kết quả đạt được**:
  ```text
  -rw------- 1 lplinh_k24cntt1 lplinh_k24cntt1 134 Oct 14 14:00 info.txt
  ```
> **Minh chứng Bài 1**: Chụp ảnh màn hình terminal VPS có dấu nhắc lệnh `lplinh_k24cntt1@...`, kết quả của `whoami`, `id`, `ls -la info.txt` và nội dung file `info.txt`.

---

## BÀI 2: Docker trên VPS (2.0 điểm)
> 💻 **VỊ TRÍ THỰC HIỆN**: **Tạo 2 file dưới đây trên MÁY CÁ NHÂN (LOCAL) trong thư mục gốc của dự án `RestAPI_Session04`**

### Yêu cầu 2.1 — Hoàn thiện Dockerfile (0.75 điểm)
> 💻 **Tạo file `Dockerfile` trong thư mục `RestAPI_Session04` trên máy local:**

```dockerfile
# -> ĐỔI NẾU KHÁC DỰ ÁN: Java 21 dùng 'eclipse-temurin:21-jre-alpine', Java 11 dùng 'eclipse-temurin:11-jre-alpine'
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

ARG JAR_FILE=build/libs/*.jar

COPY ${JAR_FILE} app.jar

# -> ĐỔI NẾU KHÁC DỰ ÁN: Đổi theo server.port trong application.properties (ví dụ 8081, 3000)
EXPOSE 8080

ENV JAVA_OPTS="-Xms256m -Xmx512m"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

---

### Yêu cầu 2.2 — Hoàn thiện Docker Compose (0.75 điểm)
> 💻 **Tạo file `docker-compose.yml` trong thư mục `RestAPI_Session04` trên máy local:**

```yaml
services:
  mysql-db:
    # -> ĐỔI NẾU KHÁC DỰ ÁN: Nếu dùng PostgreSQL thì đổi thành 'postgres:15-alpine'
    image: mysql:8.0
    container_name: restapi_mysql
    restart: always
    environment:
      # -> ĐỔI NẾU KHÁC DỰ ÁN: Đổi password và tên database đúng với application.properties của dự án mới
      MYSQL_ROOT_PASSWORD: 1234$
      MYSQL_DATABASE: db_cntt3_api_ss4
    ports:
      # -> ĐỔI NẾU KHÁC DỰ ÁN: Cổng ngoài host : Cổng container (đổi 3307:3306 nếu host bị trùng port 3306)
      - "3307:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p1234$"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: .
      dockerfile: Dockerfile
    # -> ĐỔI NẾU KHÁC DỰ ÁN: Đổi username Docker Hub và tên image theo dự án mới
    image: phuonglinhle/ptit-demo-final:latest
    container_name: restapi_backend
    restart: unless-stopped
    ports:
      # -> ĐỔI NẾU KHÁC DỰ ÁN: Port_Host : Port_Container (ví dụ "8081:8080" nếu host bị trùng port 8080)
      - "8080:8080"
    environment:
      # -> ĐỔI NẾU KHÁC DỰ ÁN: Hostname trỏ vào service 'mysql-db' (không dùng localhost), đổi tên database 'db_cntt3_api_ss4'
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql-db:3306/db_cntt3_api_ss4?createDatabaseIfNotExist=true&useSSL=false&allowPublicKeyRetrieval=true
      # -> ĐỔI NẾU KHÁC DỰ ÁN: Username/password khớp với cấu hình ở service mysql-db phía trên
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: 1234$
      SPRING_JPA_HIBERNATE_DDL_AUTO: update
    depends_on:
      mysql-db:
        condition: service_healthy
    networks:
      - app-network

volumes:
  mysql_data:
    driver: local

networks:
  app-network:
    driver: bridge
```

---

### Yêu cầu 2.3 — Kiểm tra Docker (0.5 điểm)

> **Lưu ý**:
> - Nếu máy tính cá nhân có Docker Desktop: Chạy `./gradlew bootJar` và `docker compose up -d` test thử tại chỗ.
> - Nếu máy cá nhân không có Docker Desktop: Chỉ cần tạo sẵn 2 file trên, sang Bài 4 GitHub Actions sẽ tự động build và chạy lên VPS cho bạn.

---

## BÀI 3: Git & GitHub (1.5 điểm)
> 💻 **VỊ TRÍ THỰC HIỆN**: **100% trên Terminal MÁY CÁ NHÂN (LOCAL)**  
> *(Tại thư mục `RestAPI_Session04`)*

### Yêu cầu 3.1 — Tạo Repository (0.5 điểm)

```bash
git init
git checkout -b main
git add .
git commit -m "feat: initial spring boot project RestAPI_Session04"

# -> ĐỔI NẾU KHÁC DỰ ÁN: Thay link repo GitHub tương ứng
git remote add origin https://github.com/phuonglinhle268/PTIT_CNTT1_IT209_Demo_Final.git
git branch -M main
git push -u origin main
```
- **Kết quả đạt được**: Toàn bộ mã nguồn đã có mặt trên GitHub tại nhánh `main`.

---

### Yêu cầu 3.2 — Commit và phát triển chức năng (0.5 điểm)
> 💻 **VỊ TRÍ THỰC HIỆN**: **Trên Terminal máy local**

```bash
# -> ĐỔI NẾU KHÁC DỰ ÁN: Đổi tên nhánh theo yêu cầu đề (ví dụ feature/product-count)
git checkout -b feature/employee-count
echo "Tính năng đếm nhân viên đang được phát triển" >> README.md
git add README.md
git commit -m "feat(employee): add employee count feature documentation"
git push -u origin feature/employee-count
```
- **Kết quả đạt được**: Nhánh `feature/employee-count` đã xuất hiện trên GitHub.

---

### Yêu cầu 3.3 — Xử lý Conflict (0.5 điểm)
> 💻 **VỊ TRÍ THỰC HIỆN**: **Trên Terminal máy local**

#### Bước 3.3.1: Tạo thay đổi trên nhánh `main`
```bash
git checkout main
echo "Phiên bản hệ thống: Version 1.0.0 Production" >> README.md
git add README.md
git commit -m "chore: update system version on main"
```

#### Bước 3.3.2: Tạo thay đổi xung đột trên nhánh `feature/employee-count`
```bash
git checkout feature/employee-count
echo "Phiên bản hệ thống: Version 1.1.0 Beta Employee Counter" >> README.md
git add README.md
git commit -m "chore: update system version on feature branch"
```

#### Bước 3.3.3: Merge vào `main` và đón nhận Conflict
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
Mở file `README.md` trong IntelliJ / VS Code, xóa các ký hiệu `<<<<<<<`, `=======`, `>>>>>>>` và sửa lại thành:
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
> **Minh chứng Bài 3**: Chụp ảnh màn hình terminal local hiển thị đồ thị rẽ nhánh và điểm merge của lệnh `git log --oneline --graph --all`.

---

## BÀI 4: Phát triển Chức năng & Triển khai CI/CD (4.5 điểm)

### Yêu cầu 4.1 — Phát triển API mới (1.0 điểm)
> 💻 **VỊ TRÍ THỰC HIỆN**: **Code trên máy cá nhân (LOCAL) trong project `RestAPI_Session04`**

#### 1. Entity `Employee.java` (`src/main/java/ra/demo/model/entity/Employee.java`):
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

#### 2. Repository `EmployeeRepository.java` (`src/main/java/ra/demo/repository/EmployeeRepository.java`):
```java
package ra.demo.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import ra.demo.model.entity.Employee;

@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
}
```

#### 3. Service Interface & Impl:
Interface `src/main/java/ra/demo/service/EmployeeService.java`:
```java
package ra.demo.service;

public interface EmployeeService {
    long countEmployees();
}
```

Class `src/main/java/ra/demo/service/impl/EmployeeServiceImpl.java`:
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

#### 4. Controller `EmployeeController.java` (`src/main/java/ra/demo/controller/EmployeeController.java`):
```java
package ra.demo.controller;

import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import ra.demo.service.EmployeeService;

@RestController
// -> ĐỔI NẾU KHÁC DỰ ÁN: Đổi endpoint theo đề bài yêu cầu
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

---

### Yêu cầu 4.2 — Xây dựng GitHub Actions CI/CD (1.5 điểm)

#### Bước 4.2.1: Chuẩn bị SSH Key trên VPS
> 🖥️ **VỊ TRÍ THỰC HIỆN**: **Trên Terminal VPS (user `lplinh_k24cntt1@221.121.4.132`)**

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
ssh-keygen -t rsa -b 4096 -C "github-actions-deploy" -f ~/.ssh/id_rsa_deploy -N ""
cat ~/.ssh/id_rsa_deploy.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
cat ~/.ssh/id_rsa_deploy
```

#### Bước 4.2.2: Cấu hình GitHub Actions Secrets
> 🌐 **VỊ TRÍ THỰC HIỆN**: **Trên Trình duyệt Web (GitHub)**

Vào Repo `PTIT_CNTT1_IT209_Demo_Final`: **Settings** $\rightarrow$ **Secrets and variables** $\rightarrow$ **Actions** $\rightarrow$ **New repository secret**:

- `DOCKER_USERNAME`: **`phuonglinhle`**
- `DOCKER_PASSWORD`: Token Docker Hub của bạn
- `VPS_HOST`: `221.121.4.132`
- `VPS_USER`: **`lplinh_k24cntt1`**
- `VPS_SSH_KEY`: Dán toàn bộ Private Key `id_rsa_deploy` vừa copy ở Bước 4.2.1
- `VPS_PORT`: `22`

#### Bước 4.2.3: Viết Workflow `.github/workflows/deploy.yml`
> 💻 **VỊ TRÍ THỰC HIỆN**: **Tạo file trên máy cá nhân (LOCAL) trong thư mục `RestAPI_Session04`**

Tạo file đường dẫn: `RestAPI_Session04/.github/workflows/deploy.yml`:

```yaml
name: CI/CD Pipeline - PTIT RestAPI Demo

on:
  push:
    branches: [ "main" ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: 1. Checkout mã nguồn
        uses: actions/checkout@v4

      - name: 2. Thiết lập JDK
        uses: actions/setup-java@v4
        with:
          # -> ĐỔI NẾU KHÁC DỰ ÁN: Thay '17' bằng '21' hoặc '11' tùy phiên bản Java của dự án
          java-version: '17'
          distribution: 'temurin'
          cache: gradle

      - name: 3. Cấp quyền & Build ứng dụng (JAR)
        run: |
          chmod +x ./gradlew
          ./gradlew bootJar -x test

      - name: 4. Đăng nhập Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: 5. Build và Push Docker Image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: true
          # -> ĐỔI NẾU KHÁC DỰ ÁN: Thay 'ptit-demo-final' bằng tên repo image của dự án mới
          tags: ${{ secrets.DOCKER_USERNAME }}/ptit-demo-final:latest

  deploy-to-vps:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
      - name: 1. Checkout repository
        uses: actions/checkout@v4

      - name: 2. Copy docker-compose.yml sang VPS
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          port: ${{ secrets.VPS_PORT || 22 }}
          source: "docker-compose.yml"
          # -> ĐỔI NẾU KHÁC DỰ ÁN: Đổi thư mục đích trên VPS nếu có quy định khác
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
            
            # -> ĐỔI NẾU KHÁC DỰ ÁN: Thay 'ptit-demo-final' bằng tên image mới
            docker pull ${{ secrets.DOCKER_USERNAME }}/ptit-demo-final:latest
            
            docker compose stop backend || true
            docker compose rm -f backend || true
            docker compose up -d
            
            docker image prune -f
            docker ps
```

---

### Yêu cầu 4.3 — Triển khai và kiểm chứng (1.5 điểm)

#### Bước 4.3.1: Commit và Push mã nguồn kích hoạt Pipeline
> 💻 **VỊ TRÍ THỰC HIỆN**: **Trên Terminal máy cá nhân (LOCAL)**

```bash
git add .
git commit -m "feat: implement employee count API and complete automated CI/CD pipeline"
git push origin main
```

#### Bước 4.3.2: Kiểm tra trạng thái trên GitHub Actions
> 🌐 **VỊ TRÍ THỰC HIỆN**: **Trên Trình duyệt Web (tab Actions của Repo GitHub)**

Vào tab **Actions** trên GitHub `PTIT_CNTT1_IT209_Demo_Final`: Cả 2 job `build-and-test` và `deploy-to-vps` đều hiện tích xanh ✅.

#### Bước 4.3.3: Kiểm chứng trực tiếp trên VPS
> 🖥️ **VỊ TRÍ THỰC HIỆN**: **Trên Terminal VPS (`lplinh_k24cntt1@221.121.4.132`)**

```bash
docker ps
docker logs restapi_backend --tail 50
```
- **Kết quả đạt được**: Container `restapi_backend` đang chạy (`Up`), log báo Spring Boot đã sẵn sàng nhận request trên port 8080.

#### Bước 4.3.4: Gọi API kiểm chứng tính năng bằng `curl`
> 💻 **VỊ TRÍ THỰC HIỆN**: **Trên Terminal máy cá nhân (hoặc trên VPS đều được)**

```bash
# -> ĐỔI NẾU KHÁC DỰ ÁN: Thay IP 221.121.4.132, port 8080 và endpoint theo đề bài
curl -X GET http://221.121.4.132:8080/api/employees/count
```
- **Kết quả ban đầu**: `0`

Chèn thử 1 bản ghi vào MySQL trên VPS:
```bash
# -> ĐỔI NẾU KHÁC DỰ ÁN: Đổi mật khẩu, tên database, tên bảng và các cột tương ứng
docker exec -it restapi_mysql mysql -uroot -p'1234$' -e "USE db_cntt3_api_ss4; INSERT INTO employees (full_name, email, department) VALUES ('Lê Phương Linh', 'linh@ptit.edu.vn', 'CNTT1');"
```

Gọi lại lệnh `curl`:
```bash
curl -X GET http://221.121.4.132:8080/api/employees/count
```
- **Kết quả đạt được**: `1`
> **Minh chứng Bài 4**: Chụp ảnh tab GitHub Actions xanh 100% kèm ảnh chụp terminal chạy lệnh `curl http://221.121.4.132:8080/api/employees/count` trả về `1`.

---

## Bảng tổng hợp Troubleshooting & Lỗi thường gặp

| Lỗi gặp phải | Nguyên nhân | Cách khắc phục triệt để |
| :--- | :--- | :--- |
| `bash: docker: command not found` | VPS mới chưa được cài đặt Docker. | Chạy lệnh `curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh`. |
| `permission denied while trying to connect to Docker daemon` | User chưa có quyền trong nhóm `docker` hoặc chưa reload session. | Chạy `sudo usermod -aG docker lplinh_k24cntt1` sau đó gõ lệnh `newgrp docker`. |
| `Host key verification failed` trong GitHub Actions | SSH Action chưa tin cậy host key của VPS. | Thêm param `fingerprint` hoặc action `appleboy/ssh-action` tự động xử lý khi khai báo đủ host, user và key. |
| `Communications link failure` khi Spring Boot khởi động | Container MySQL khởi động chậm hơn Spring Boot. | Dùng `healthcheck` trên MySQL và `condition: service_healthy` ở `depends_on` trong `docker-compose.yml`. |
| `Permission denied: ./gradlew` trong GitHub Actions | File `gradlew` mất cờ thực thi khi push lên Git từ Windows. | Thêm step `run: chmod +x ./gradlew` trước khi chạy `./gradlew bootJar`. |
| `Load key "...": error in libcrypto` | SSH Private Key copy vào GitHub Secret bị thiếu dòng header/footer hoặc bị ngắt dòng sai. | Copy chính xác từ `-----BEGIN OPENSSH PRIVATE KEY-----` đến `-----END OPENSSH PRIVATE KEY-----` kèm đầy đủ ký tự xuống dòng. |
| MySQL lỗi `Access denied for user 'root'@'...'` | Mật khẩu khai báo trong biến môi trường không khớp giữa MySQL và Backend. | Đảm bảo `MYSQL_ROOT_PASSWORD` trong mysql-db bằng chính xác `SPRING_DATASOURCE_PASSWORD` trong backend. |
| `ports are not available: listen tcp 0.0.0.0:3306: bind` | Cổng 3306 trên máy host đã bị MySQL local chiếm dụng. | Sửa trong `docker-compose.yml` cổng MySQL thành `"3307:3306"`. |
