 # Bài 4 - Khai thác N8N để tự động đăng bài lên WordPress

**Môn:** Phát triển ứng dụng với mã nguồn mở (TEE0421)  
**Lớp:** 58KTPM  
**Sinh viên:** Lương Quang Hà - K225480106010  
**Deadline:** 23h59 ngày 25/05/2026

---

## Kiến trúc hệ thống
Telegram Bot → N8N Trigger → Google Gemini AI → JavaScript Code → WordPress Post
<!-- Chèn ảnh: sơ đồ workflow N8N (4 node nối nhau) -->

---

## 1. Chuẩn bị môi trường

### 1.1 Yêu cầu
- Ubuntu Server 24.04 (Hyper-V)
- Docker & Docker Compose đã cài sẵn
- Domain: `luongquangha.io.vn` (quản lý qua Cloudflare)
- Tài khoản Cloudflare Zero Trust

### 1.2 SSH vào Ubuntu Server

```bash
ssh luongha@172.27.243.184
```
<img width="979" height="513" alt="image" src="https://github.com/user-attachments/assets/cc89c56f-29b9-473a-bf1d-a6ddc9ba4881" />


---

## 2. Docker Compose - 5 Services

### 2.1 Tạo thư mục và file cấu hình

```bash
mkdir -p ~/wordpress
cd ~/wordpress
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d540d80b-d7ae-4578-8ada-50e62ae522ed" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f948a9e7-14f5-4a1d-b3cb-d519436c4475" />


### 2.2 Nội dung file docker-compose.yml

```yaml
services:
  mariadb:
    image: mariadb:latest
    container_name: wp_mariadb
    restart: always
    environment:
      TZ: "Asia/Ho_Chi_Minh"
      MYSQL_ROOT_PASSWORD: rootpass123
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: wp_pass123
    volumes:
      - mariadb_data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: wp_phpmyadmin
    restart: always
    environment:
      PMA_HOST: wp_mariadb
      PMA_ARBITRARY: 1
    depends_on:
      - mariadb

  wordpress:
    image: wordpress:latest
    container_name: wp_app
    restart: always
    environment:
      WORDPRESS_DB_HOST: wp_mariadb:3306
      WORDPRESS_DB_NAME: wordpress_db
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: wp_pass123
    volumes:
      - wordpress_data:/var/www/html
    depends_on:
      - mariadb

  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: wp_cloudflared
    restart: always
    command: tunnel --no-autoupdate run --token <CLOUDFLARE_TUNNEL_TOKEN>
    depends_on:
      - wordpress
      - phpmyadmin
      - n8n

  n8n:
    image: n8nio/n8n:latest
    container_name: wp_n8n
    restart: always
    environment:
      TZ: "Asia/Ho_Chi_Minh"
      WEBHOOK_URL: "https://k58-n8n.luongquangha.io.vn/"
      N8N_HOST: "k58-n8n.luongquangha.io.vn"
      N8N_PORT: 5678
      N8N_PROTOCOL: https
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  mariadb_data:
  wordpress_data:
  n8n_data:
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/04d17078-1580-43b1-ba1c-884e54c5d58d" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4b54d192-82a6-48e2-9661-63594901f6d2" />

# luongha@ubuntusever:~/wordpress$ cat docker-compose.yml:

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/07f0dd07-52bf-4799-b301-a279a31e609f" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4d6d2f60-b464-4433-9fe5-431bd9b356bc" />

> **Lưu ý:** Thay `<CLOUDFLARE_TUNNEL_TOKEN>` bằng token thực lấy từ Cloudflare Dashboard.

### 2.3 Kéo images và chạy services

```bash
# Dọn dẹp disk nếu cần
docker system prune -af

# Kéo tất cả images
docker-compose pull

# Khởi động tất cả services
docker-compose up -d

# Kiểm tra trạng thái
docker-compose ps
```
kết quả docker-compose pull - 5 images done
<img width="981" height="514" alt="image" src="https://github.com/user-attachments/assets/d9035e6b-03f5-401f-9e97-6e3eac2fd351" />

kết quả docker-compose ps - 5 services đều Up
<img width="981" height="516" alt="image" src="https://github.com/user-attachments/assets/31ad5e53-f116-45c4-882e-4755ea2e0bd4" />


---

## 3. Cấu hình Cloudflare Tunnel

### 3.1 Lấy Tunnel Token
1. Vào `https://dash.cloudflare.com` → Zero Trust → Networks → Connectors
2. Bấm vào tunnel `luongquangha-tunnel`
3. Bấm **Add a connector** → chọn **Docker**
4. Copy token từ lệnh hiển thị

Cloudflare Tunnel HEALTHY và Connected 
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/552e0f3b-baef-48fe-a507-e7589cd20291" />

### 3.2 Thêm 3 Public Hostname Routes

Vào tab **Published application routes** → **Add a published application route**:

| STT | Subdomain | Domain | Service |
|-----|-----------|--------|---------|
| 1 | `k58-wp` | `luongquangha.io.vn` | `http://wp_app:80` |
| 2 | `k58-pma` | `luongquangha.io.vn` | `http://wp_phpmyadmin:80` |
| 3 | `k58-n8n` | `luongquangha.io.vn` | `http://wp_n8n:5678` |

 Cloudflare Dashboard hiển thị 3 routes đã thêm 
<img width="1910" height="1079" alt="image" src="https://github.com/user-attachments/assets/547a042a-a65d-43a6-a898-aa25ba103365" />

---

## 4. Cài đặt WordPress

### 4.1 Kiểm tra DB trước khi cài

Truy cập `https://k58-pma.luongquangha.io.vn`:
- Máy chủ: `wp_mariadb`
- Tài khoản: `root`
- Mật khẩu: `rootpass123`

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ee1637df-bb78-44a4-be5e-150c0f8f6508" />


### 4.2 Cài đặt WordPress

Truy cập `https://k58-wp.luongquangha.io.vn` và điền thông tin:
- Tiêu đề: `Blog của Lương Quang Hà`
- Tên người dùng: `luongha`
- Email: `hluongquang764@gmail.com`

<img width="1456" height="819" alt="image" src="https://github.com/user-attachments/assets/1a7cd488-71ab-4bf5-8bec-180c061dfb13" />


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/72a089a8-bb80-4a3f-92ba-f2957a8f97ce" />


### 4.3 Kiểm tra DB sau khi cài

<img width="1911" height="1075" alt="image" src="https://github.com/user-attachments/assets/9abea158-3b57-477a-aacc-c9a53f4e0a14" />


---

## 5. Tạo 2 bài viết thủ công

### Bài viết 1: Giới thiệu bản thân

Vào **Bài viết → Thêm mới**, tạo bài giới thiệu bản thân gồm: thông tin cá nhân, sở thích, mục tiêu nghề nghiệp.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5bd5a5b0-f7be-4c8a-ad44-300261971832" />


### Bài viết 2: Kiến thức môn học

Tạo bài viết tổng hợp kiến thức đã học trong môn TEE0421: Docker, Cloudflare Tunnel, Ubuntu Server, WordPress, N8N.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e6fa7b59-b455-4ca1-9e7b-136184dc68ba" />


---

## 6. Cấu hình N8N

### 6.1 Tạo tài khoản admin

Truy cập `https://k58-n8n.luongquangha.io.vn`:
- Email: `hluongquang764@gmail.com`
- First Name: `Quang Ha`
- Last Name: `Luong`
- Password: `********`

<img width="1456" height="819" alt="image" src="https://github.com/user-attachments/assets/646957fb-3b12-41e8-8f64-0890895a40e1" />


### 6.2 Kích hoạt License Key

Bấm **Send me a free license key** → kiểm tra email → copy key → vào **Settings → Usage and plan → Enter activation key**.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/bc6e19de-dab0-4ff6-a128-1347b8e496cd" />


### 6.3 Tạo Telegram Bot

1. Mở Telegram → tìm `@BotFather` → gõ `/newbot`
2. Đặt tên: `K58 WordPress Bot`
3. Đặt username: `k58wordpress_bot`
4. Copy token nhận được
5. Mở bot mới và nhắn tin bất kỳ (bước quan trọng!)
<img width="870" height="1885" alt="image" src="https://github.com/user-attachments/assets/a48be289-52c8-459e-bd2d-fb85a203599e" />


### 6.4 Lấy Google Gemini API Key

1. Truy cập `https://aistudio.google.com/api-keys`
2. Tạo project mới
3. Bấm **Tạo khóa** → copy API Key

<img width="1456" height="824" alt="image" src="https://github.com/user-attachments/assets/85ec5279-07ae-4c66-8265-1083e8b6ef44" />


### 6.5 Tạo Application Password trong WordPress

Vào `https://k58-wp.luongquangha.io.vn/wp-admin` → **Tài khoản → Hồ sơ** → kéo xuống **Mật khẩu ứng dụng** → nhập tên `n8n` → bấm **Thêm mật khẩu ứng dụng** → copy chuỗi 24 ký tự.

<img width="1456" height="819" alt="image" src="https://github.com/user-attachments/assets/66d69573-50fa-40a4-9b71-34977f4e0efd" />


---

## 7. Xây dựng N8N Workflow

### 7.1 Sơ đồ workflow
[Telegram Trigger] → [Google Gemini - Message a model] → [Code in JavaScript] → [WordPress - Create a post]
### 7.2 Node 1: Telegram Trigger

- Trigger On: **Message**
- Credential: nhập token bot Telegram

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2cbd5df5-86ac-440b-b716-7bc8b11f4a92" />


### 7.3 Node 2: Google Gemini - Message a model

- Model: `models/gemini-3-flash-preview`
- Prompt (Expression):
- {{ $json.message.text }}. Hãy tạo một bài viết WordPress hoàn chỉnh với tiêu đề và nội dung dựa trên chủ đề trên. Trả về kết quả dưới dạng JSON với format: {"post_title": "tiêu đề bài viết", "post_content": "nội dung HTML của bài viết"}. Chỉ trả về JSON, không có text nào khác.
- Output Content as JSON: **ON**

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/bec67066-0df0-4709-a14d-1b1371aefadc" />


### 7.4 Node 3: Code in JavaScript

```javascript
const input = $input.first().json;

let content = input.content;
if (typeof content !== 'string') {
  content = JSON.stringify(content);
}

const title = 'Bài viết: ' + new Date().toLocaleDateString('vi-VN');

return [{ json: { title, content } }];
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7c52b5b6-bb5a-4252-8053-58a4869c8a33" />


### 7.5 Node 4: WordPress - Create a post

- Credential: WordPress account (URL + username + Application Password)
- Title: `{{ $json.title }}`
- Content: `{{ $json.content }}`
- Status: **Publish**
- Ignore SSL Issues: **ON**
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2e64b9d2-d71b-4394-b8af-f4bc7c614750" />


### 7.6 Publish Workflow

Bấm **Publish** góc trên phải để kích hoạt workflow.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/fbf0b836-a596-4e6a-93fe-fad8829a6c6f" />


---

## 8. Kết quả demo

### 8.1 Nhắn tin với bot Telegram
<img width="870" height="1885" alt="image" src="https://github.com/user-attachments/assets/b57d0cc8-9cf1-4ac6-b790-ea743b9e4a1a" />


### 8.2 N8N Execution thành công

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/24c9de1e-f7ae-43b7-a81a-58a58833f879" />


### 8.3 Bài viết tự động xuất hiện trên WordPress

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4123aac0-8987-45a2-a1d8-afb2bdea9fd7" />


---

## Nhận xét

Bài tập giúp hiểu rõ cách kết hợp các công cụ mã nguồn mở trong thực tế:

- **Docker Compose** giúp quản lý nhiều service phức tạp chỉ với 1 file cấu hình
- **Cloudflare Tunnel** cho phép public dịch vụ nội bộ ra internet an toàn, không cần mở port
- **N8N** là công cụ automation mạnh mẽ, kết nối được hầu hết các dịch vụ phổ biến
- **Google Gemini AI** tích hợp dễ dàng qua API, sinh nội dung chất lượng cao
- **WordPress REST API** + Application Password cho phép đăng bài từ bên ngoài an toàn

**Kết quả đạt được:** Chỉ cần nhắn 1 tin nhắn Telegram, trong vòng 15-20 giây bài viết HTML đầy đủ sẽ tự động xuất hiện trên WordPress mà không cần thao tác thủ công.
