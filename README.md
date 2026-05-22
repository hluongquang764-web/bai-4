# Bài 4 - Khai thác N8N để tự động đăng bài lên WordPress

**Môn:** Phát triển ứng dụng với mã nguồn mở (TEE0421)  
**Lớp:** 58KTPM  
**Sinh viên:** Lương Quang Hà - K225480106010

---

## Kiến trúc hệ thống
Telegram Bot → N8N Trigger → Google Gemini AI → JavaScript Code → WordPress Post
<!-- Chèn ảnh: sơ đồ workflow N8N (4 node nối nhau) -->

---

## 1. Docker Compose - 5 Services

File `docker-compose.yml` gồm: MariaDB, phpMyAdmin, WordPress, Cloudflared, N8N.

<!-- Chèn ảnh: nội dung file docker-compose.yml -->

```bash
docker-compose pull
docker-compose up -d
docker-compose ps
```

<!-- Chèn ảnh: kết quả docker-compose ps - 5 services đều Up -->

---

## 2. Cloudflare Tunnel - 3 Subdomain

| Subdomain | Service |
|-----------|---------|
| k58-wp.luongquangha.io.vn | WordPress |
| k58-pma.luongquangha.io.vn | phpMyAdmin |
| k58-n8n.luongquangha.io.vn | N8N |

<!-- Chèn ảnh: Cloudflare Dashboard - Published application routes với 3 subdomain -->

---

## 3. phpMyAdmin - DB trước và sau khi cài WordPress

<!-- Chèn ảnh: DB trống trước khi cài WordPress (chỉ có form tạo bảng, không có bảng nào) -->

<!-- Chèn ảnh: DB sau khi cài WordPress (12 bảng wp_*) -->

---

## 4. Cài đặt WordPress

Truy cập `https://k58-wp.luongquangha.io.vn` để cài đặt.

<!-- Chèn ảnh: trang cài đặt WordPress điền thông tin -->

<!-- Chèn ảnh: trang quản trị WordPress sau khi cài xong -->

---

## 5. Bài viết thủ công trên WordPress

### Bài 1: Giới thiệu bản thân

<!-- Chèn ảnh: bài viết "Giới thiệu về bản thân - Lương Quang Hà" đã xuất bản -->

### Bài 2: Kiến thức đã học

<!-- Chèn ảnh: bài viết "Những kiến thức học được từ môn Phát triển ứng dụng với mã nguồn mở" đã xuất bản -->

---

## 6. Cấu hình N8N

### 6.1 Tạo tài khoản & Kích hoạt License

<!-- Chèn ảnh: màn hình đăng ký tài khoản N8N -->

<!-- Chèn ảnh: thông báo "License activated - Your Registered Community Edition has been successfully activated" -->

### 6.2 Tạo Telegram Bot

Chat với @BotFather để tạo bot `@k58wordpress_bot`.

<!-- Chèn ảnh: BotFather trả về token bot mới -->

### 6.3 Lấy Google Gemini API Key

Tạo API Key tại `https://aistudio.google.com/api-keys`.

<!-- Chèn ảnh: trang tạo Gemini API Key thành công -->

### 6.4 Workflow N8N hoàn chỉnh

4 node theo thứ tự:
1. **Telegram Trigger** - nhận tin nhắn từ bot
2. **Google Gemini** - sinh nội dung HTML từ prompt
3. **Code in JavaScript** - xử lý output từ Gemini
4. **WordPress Create a post** - đăng bài tự động

<!-- Chèn ảnh: workflow N8N với 4 node đã Published -->

### 6.5 Cấu hình Telegram Credential

<!-- Chèn ảnh: "Connection tested successfully" trong Telegram credential -->

### 6.6 Cấu hình WordPress Credential

Lấy Application Password tại: WP Admin → Tài khoản → Hồ sơ → Mật khẩu ứng dụng.

<!-- Chèn ảnh: tạo Application Password "n8n" trong WordPress -->

---

## 7. Kết quả - Tự động đăng bài qua Telegram

### 7.1 Nhắn tin với bot

<!-- Chèn ảnh: điện thoại nhắn tin với @k58wordpress_bot -->

### 7.2 N8N Execution thành công

<!-- Chèn ảnh: Executions tab - "Succeeded" với 4 node đều tích xanh -->

### 7.3 Bài viết tự động xuất hiện trên WordPress

<!-- Chèn ảnh: trang WordPress hiển thị bài viết mới được đăng tự động -->

---

## Nhận xét

Bài tập giúp hiểu rõ cách kết hợp các công cụ mã nguồn mở:
- **Docker** để đóng gói và quản lý nhiều service
- **Cloudflare Tunnel** để public dịch vụ an toàn
- **N8N** để tự động hóa workflow không cần code phức tạp
- **Google Gemini AI** để sinh nội dung thông minh
- **WordPress REST API** để tích hợp với hệ thống bên ngoài

Kết quả đạt được: chỉ cần nhắn 1 tin Telegram, bài viết HTML đầy đủ sẽ tự động xuất hiện trên WordPress trong vòng 15-20 giây.
