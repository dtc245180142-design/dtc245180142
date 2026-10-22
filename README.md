# BÁO CÁO BÀI TẬP LỚN TRIỂN KHAI VÀ QUẢN TRỊ HỆ THỐNG PHẦN MỀM

- **Học phần:** Triển khai và Quản trị Hệ thống Phần mềm
- **Đề tài 19:** Website Tin tức / Cổng thông tin (WordPress, MySQL, phpMyAdmin, Nginx HTTPS, Prometheus, Grafana, Loki)
- **Sinh viên thực hiện:** Hoàng Đình Gia Đạt
- **Mã số sinh viên (MSSV):** DTC245180142
- **Tài khoản GitHub:** dtc245180142-design
- **Repository:** https://github.com/dtc245180142-design/dtc245180142

---

## 1. Tổng quan Kiến trúc Hệ thống

Hệ thống được thiết kế và triển khai trên nền tảng **Docker Compose** bao gồm 9 dịch vụ (microservices) phối hợp chặt chẽ:

1. **Website Tin tức (WordPress):** Cổng thông tin cho phép quản trị viên đăng tải bài viết, phân loại chuyên mục và người dùng đọc tin tức.
2. **Cơ sở dữ liệu (MySQL 8.0):** Lưu trữ toàn bộ dữ liệu bài viết, tài khoản và cấu hình hệ thống.
3. **Quản trị CSDL (phpMyAdmin):** Công cụ quản lý cơ sở dữ liệu qua giao diện Web trực quan.
4. **Proxy biên & Cân bằng tải (Nginx):** Reverse Proxy chịu trách nhiệm tiếp nhận lưu lượng, thiết lập kết nối mã hóa HTTPS và bổ sung các Header bảo vệ.
5. **Thu thập chỉ số tài nguyên (cAdvisor):** Thu thập thông số phần cứng, CPU, RAM, Network của từng container.
6. **Máy chủ lưu trữ số liệu (Prometheus):** Quản lý cơ sở dữ liệu chuỗi thời gian (TSDB) các chỉ số hoạt động.
7. **Trực quan hóa (Grafana):** Hiển thị hệ thống đồ thị Dashboard giám sát container và hệ thống.
8. **Máy chủ Log tập trung (Loki):** Lưu trữ và lập chỉ mục nhật ký hoạt động của các dịch vụ.
9. **Thu thập Log (Promtail):** Thu thập toàn bộ log container từ Docker daemon gửi về Loki.

---

## 2. Hướng dẫn Cài đặt và Vận hành Hệ thống

### Bước 1: Sao chép (Clone) mã nguồn về máy ảo Ubuntu
```bash
git clone https://github.com/dtc245180142-design/dtc245180142.git
cd dtc245180142
```

### Bước 2: Tạo chứng chỉ số SSL tự ký cho Nginx (HTTPS)
```bash
mkdir -p nginx/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/ssl/nginx.key -out nginx/ssl/nginx.crt \
  -subj "/C=VN/ST=HCM/L=HCM/O=Student/CN=localhost"
```

### Bước 3: Khởi chạy toàn bộ hệ thống bằng Docker Compose
```bash
docker compose up -d
```

### Bước 4: Kiểm tra trạng thái hoạt động của các dịch vụ
```bash
docker compose ps
```
*(Đảm bảo tất cả 9 dịch vụ đều ở trạng thái `Up` hoặc `running`).*

---

## 3. Danh sách Dịch vụ và Địa chỉ Truy cập

| Tên Dịch Vụ | Chức Năng | Địa Chỉ Truy Cập | Tài Khoản Mặc Định |
| :--- | :--- | :--- | :--- |
| **Website Tin tức** | Cổng thông tin WordPress | `https://192.168.142.129` | Khởi tạo tài khoản Admin qua giao diện |
| **phpMyAdmin** | Quản trị CSDL MySQL | `http://192.168.142.129:8080` (hoặc `8081`) | `news_admin` / `NewsPortalSecurePass2026!@#` |
| **Prometheus** | Kiểm tra Metrics & Targets | `http://192.168.142.129:9090` (hoặc `9091`) | Không yêu cầu |
| **Grafana** | Dashboard giám sát hệ thống | `http://192.168.142.129:3000` (hoặc `3001`) | `admin` / `admin` |
| **Loki API** | Endpoint nhận và truy vấn Log | `http://192.168.142.129:3100` | Không yêu cầu |

---

## 4. Các câu truy vấn LogQL mẫu (Centralized Logging)

Hệ thống hỗ trợ phân tích log tập trung thông qua ngôn ngữ **LogQL** trên giao diện Grafana Explore:

- **Truy vấn 1: Giám sát toàn bộ Access Log qua Nginx Reverse Proxy**
  ```logql
  {job="docker-logs"} |= "de19_nginx"
  ```
- **Truy vấn 2: Theo dõi hoạt động và truy vấn của Website WordPress**
  ```logql
  {job="docker-logs"} |= "de19_wordpress"
  ```
- **Truy vấn 3: Bắt các thông báo lỗi và cảnh báo từ Cơ sở dữ liệu MySQL**
  ```logql
  {job="docker-logs"} |= "de19_mysql"
  ```

---

## 5. Các biện pháp Bảo mật (System Hardening) đã áp dụng

Hệ thống đã triển khai đầy đủ các biện pháp an ninh mạng theo yêu cầu môn học:
1. **Cô lập phân vùng mạng (Network Isolation):** Phân chia làm 3 mạng riêng biệt `frontend`, `backend`, `monitoring`. CSDL MySQL chỉ nằm trong mạng `backend`, hoàn toàn không mở cổng ra ngoài máy chủ để chống dò quét.
2. **Bảo vệ bằng HTTPS & Security Headers:** Nginx chuyển hướng tự động HTTP sang HTTPS, bổ sung các tiêu đề bảo mật chống tấn công Clickjacking (`X-Frame-Options: SAMEORIGIN`), chống thực thi sai định dạng file (`X-Content-Type-Options: nosniff`) và kích hoạt bộ lọc chống XSS (`X-XSS-Protection`).
3. **Chính sách Mật khẩu mạnh & Bảo mật biến môi trường:** Toàn bộ mật khẩu root và user được tách biệt trong tệp `.env`, không bị hardcode trong mã nguồn, có độ phức tạp cao kèm ký tự đặc biệt.
4. **Phân quyền tối thiểu (Least Privilege):** Ứng dụng Web chỉ kết nối tới CSDL thông qua tài khoản giới hạn quyền `news_admin`, không sử dụng tài khoản `root`.

---

## 6. Lịch sử Các Commit (Git History)
- **Commit 1:** Triển khai ứng dụng Web tin tức WordPress, MySQL, phpMyAdmin và Nginx HTTPS Reverse Proxy.
- **Commit 2:** Tích hợp hệ thống giám sát container, web, database bằng Prometheus và Grafana.
- **Commit 3:** Triển khai hệ thống log tập trung Loki, Promtail và áp dụng các biện pháp Hardening.
