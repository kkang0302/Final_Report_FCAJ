---
title : "Cơ sở dữ liệu RDS"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

# Triển khai lớp Cơ sở dữ liệu Amazon RDS (PostgreSQL)

Mỗi microservice trong hệ thống CourShare đều cần một cơ sở dữ liệu quan hệ riêng biệt. Để tiết kiệm chi phí thử nghiệm thực tế (tránh phải khởi chạy 5 instance RDS riêng biệt gây tốn kém hàng trăm USD/tháng), chúng ta thiết kế một kiến trúc **Multi-schema** chạy trên cùng một database instance **Amazon RDS PostgreSQL** chung, đặt trong phân vùng Database Subnet an toàn.

## Bước 1: Tạo DB Subnet Group
Để Amazon RDS có thể chạy dự phòng, AWS yêu cầu database phải liên kết với ít nhất 2 Subnets thuộc 2 Availability Zones khác nhau.
1. Truy cập **RDS Console** -> **Subnet groups** -> **Create DB Subnet Group**.
2. Đặt tên: `courshare-db-subnet-group`.
3. VPC: Chọn `courshare-vpc`.
4. Add subnets: Chọn AZs `us-east-1a`, `us-east-1b` và chọn đúng 2 Database Subnets: `courshare-db-subnet-1` (`10.0.20.0/24`) và `courshare-db-subnet-2` (`10.0.21.0/24`).
5. Click **Create**.

## Bước 2: Khởi tạo Database Instance RDS
1. Vào **RDS Console** -> **Databases** -> **Create database**.
2. Chọn phương thức: **Standard create**.
3. Engine options: Chọn **PostgreSQL**.
4. Templates: Chọn **Free Tier** (hoặc Dev/Test).
5. Settings:
   - DB instance identifier: `courshare-rds`.
   - Master username: `courshare`.
   - Master password: `courshare-secure-password`.
6. Instance configuration:
   - DB instance class: Chọn **db.t4g.micro** (chi phí cực kỳ rẻ, tối ưu cho thử nghiệm).
7. Connectivity:
   - Virtual Private Cloud (VPC): Chọn `courshare-vpc`.
   - DB Subnet Group: Chọn `courshare-db-subnet-group`.
   - Public access: **No** (Bảo mật tuyệt đối, không cho phép truy cập trực tiếp từ Internet).
   - VPC Security Group: Chọn **Choose existing** -> Bỏ default SG và chọn đúng `courshare-rds-sg`.
8. Additional configuration:
   - Initial database name: `courshare_db`.
9. Click **Create database**.

## Bước 3: Tạo 5 database schema bằng Bastion Host
Vì RDS nằm trong Database Subnet không có IP public, chúng ta phải SSH vào **NAT Instance** (ở Public Subnet) đóng vai trò làm Bastion Host để kết nối và quản trị database:

1. SSH vào NAT instance từ máy local.
2. Cài đặt PostgreSQL client trên NAT instance:
   ```bash
   sudo dnf install postgresql15 -y
   ```
3. Kết nối vào RDS instance thông qua Endpoint được cấp:
   ```bash
   psql -h courshare-rds.xxxxxx.us-east-1.rds.amazonaws.com -U courshare -d courshare_db
   ```
4. Thực thi các câu lệnh SQL để khởi tạo 5 schemas riêng cho 5 microservices:
   ```sql
   CREATE SCHEMA identity_db;
   CREATE SCHEMA course_db;
   CREATE SCHEMA enrollment_db;
   CREATE SCHEMA learning_db;
   CREATE SCHEMA payment_db;
   ```

## Bước 4: Kết nối ECS Tasks vào RDS
Cập nhật biến môi trường kết nối database trong ECS Task Definition cho từng service:

* **Identity Service (Java Spring Boot):**
  - Biến môi trường: `SPRING_DATASOURCE_URL = jdbc:postgresql://courshare-rds.xxxxxx.us-east-1.rds.amazonaws.com:5432/courshare_db?currentSchema=identity_db`
* **Course Service (Java Spring Boot):**
  - Biến môi trường: `SPRING_DATASOURCE_URL = jdbc:postgresql://courshare-rds.xxxxxx.us-east-1.rds.amazonaws.com:5432/courshare_db?currentSchema=course_db`
* **Enrollment Service (Node.js/Express):**
  - Biến môi trường: `DATABASE_URL = postgresql://courshare:courshare-secure-password@courshare-rds.xxxxxx.us-east-1.rds.amazonaws.com:5432/courshare_db?schema=enrollment_db`
* **Learning Service (Node.js/Express):**
  - Biến môi trường: `DATABASE_URL = postgresql://courshare:courshare-secure-password@courshare-rds.xxxxxx.us-east-1.rds.amazonaws.com:5432/courshare_db?schema=learning_db`
* **Payment Service (Node.js/Express):**
  - Biến môi trường: `DATABASE_URL = postgresql://courshare:courshare-secure-password@courshare-rds.xxxxxx.us-east-1.rds.amazonaws.com:5432/courshare_db?schema=payment_db`

*(Sau khi redeploy, các ECS Tasks sẽ tự động kết nối qua mạng nội bộ VPC, tự chạy script khởi tạo các bảng tương ứng trong schema của mình).*

<!-- TODO: chèn screenshot - [Màn hình Amazon RDS Console hiển thị database instance courshare-rds với trạng thái Available và Endpoint kết nối] -->

<!-- TODO: chèn screenshot - [Màn hình DBeaver kết nối vào database hiển thị danh sách các bảng như users, courses, enrollments, payments được tạo độc lập trong 5 schema khác nhau] -->
