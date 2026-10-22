# ĐỒ ÁN TRIỂN KHAI VÀ QUẢN TRỊ HỆ THỐNG PHẦN MỀM
- **Đề tài 19:** Website Tin tức / Cổng thông tin
- **Sinh viên thực hiện:** [Hoàng Đình Gia Đạt]
- **Mã số sinh viên (MSSV):** [dtc245180142]
- **Stack công nghệ:** WordPress, MySQL 8.0, phpMyAdmin, Nginx HTTPS, Prometheus, cAdvisor, Grafana, Loki, Promtail.

## Hướng dẫn cài đặt và khởi chạy:
## 1. Yêu cầu hệ thống
- Hệ điều hành: Ubuntu 22.04 LTS trở lên.
- Đã cài đặt Docker Engine và Docker Compose plugin.
- Mở các cổng: 80, 443, 8080 (hoặc 8081), 9090, 3000 (hoặc 3001).
---
## 2. Hướng dẫn cài đặt và khởi chạy hệ thống
### Bước 1: Clone mã nguồn về máy
```bash
git clone https://github.com/dtc245180142-design/dtc245180142.git
cd dtc245180142
###Bước 2: Tạo chứng chỉ SSL tự ký cho Nginx (HTTPS)
'''bash
mkdir -p nginx/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/ssl/nginx.key -out nginx/ssl/nginx.crt \
  -subj "/C=VN/ST=HCM/L=HCM/O=Student/CN=localhost"
###Bước 3: Khởi chạy toàn bộ hệ thống bằng Docker Compose
'''bash
docker compose up -d
###Bước 4: Kiểm tra trạng thái các container
'''bash
docker compose ps
