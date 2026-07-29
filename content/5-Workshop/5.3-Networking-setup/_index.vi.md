---
title : "Thiết lập mạng VPC"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

# Thiết lập hạ tầng mạng VPC cho CourShare

Hạ tầng mạng VPC vững chắc và bảo mật là bước nền tảng để triển khai cụm Microservices chạy trên ECS Fargate. Chúng ta sẽ phân chia mạng thành 3 phân lớp subnet khác nhau trên 2 Availability Zones (AZs) để đảm bảo High Availability và an toàn dữ liệu.

## Bước 1: Khởi tạo VPC
1. Đăng nhập vào AWS Console, tìm kiếm dịch vụ **VPC**.
2. Nhấp vào **Create VPC**.
3. Chọn **VPC only** (chúng ta tự thiết lập các thành phần thủ công để hiểu luồng mạng).
4. Tên VPC: `courshare-vpc`.
5. IPv4 CIDR block: `10.0.0.0/16`.
6. Nhấp vào **Create VPC**.

## Bước 2: Tạo các Subnets (Phân hoạch IP)
Chúng ta sẽ tạo 6 subnets trên 2 AZs (`us-east-1a` và `us-east-1b`):

1. **2 Public Subnets** (cho ALB và NAT Instance):
   - `courshare-public-subnet-1` | CIDR: `10.0.1.0/24` | AZ: `us-east-1a`
   - `courshare-public-subnet-2` | CIDR: `10.0.2.0/24` | AZ: `us-east-1b`
2. **2 Private Subnets** (cho ECS Tasks chạy các microservices và workers):
   - `courshare-private-subnet-1` | CIDR: `10.0.10.0/24` | AZ: `us-east-1a`
   - `courshare-private-subnet-2` | CIDR: `10.0.11.0/24` | AZ: `us-east-1b`
3. **2 Database Subnets** (cho RDS PostgreSQL):
   - `courshare-db-subnet-1` | CIDR: `10.0.20.0/24` | AZ: `us-east-1a`
   - `courshare-db-subnet-2` | CIDR: `10.0.21.0/24` | AZ: `us-east-1b`

*(Đảm bảo đã kích hoạt thuộc tính "Auto-assign public IPv4 address" cho các Public Subnet).*

## Bước 3: Tạo Internet Gateway (IGW)
1. Trong menu bên trái VPC Console, chọn **Internet Gateways** -> **Create Internet Gateway**.
2. Đặt tên: `courshare-igw` và nhấp vào **Create**.
3. Chọn IGW vừa tạo -> **Actions** -> **Attach to VPC** -> Chọn `courshare-vpc` -> Nhấp **Attach**.

## Bước 4: Triển khai NAT Instance (Tối ưu chi phí)
Thay vì sử dụng AWS NAT Gateway (chi phí cố định khoảng $32/tháng), chúng ta sử dụng một máy ảo EC2 siêu nhỏ làm NAT:

1. Vào **EC2 Console** -> **Launch Instance**.
2. Tên instance: `courshare-nat-instance`.
3. Hệ điều hành: Chọn **Amazon Linux 2023 AMI**.
4. Instance Type: Chọn **t4g.nano** (tiết kiệm chi phí, chỉ khoảng $3.2/tháng).
5. Network:
   - VPC: `courshare-vpc`
   - Subnet: `courshare-public-subnet-1`
   - Auto-assign Public IP: Enable
6. Chọn key pair và tạo Security Group cho phép lưu lượng đi ra. Kích hoạt thuộc tính IP Forwarding trong Linux của NAT instance.
7. **QUAN TRỌNG:** Sau khi máy ảo khởi chạy thành công, nhấp chuột phải vào NAT instance -> **Networking** -> **Change source/destination checking**. Chọn **Disable** (Bắt buộc phải tắt để máy ảo có thể chuyển tiếp gói tin từ Private Subnet ra ngoài).

## Bước 5: Cấu hình Route Tables (Bảng định tuyến)
Chúng ta tạo 3 bảng định tuyến:

1. **Public Route Table (`courshare-public-rt`):**
   - Assocations: Gắn với `courshare-public-subnet-1` và `2`.
   - Routes: `0.0.0.0/0` -> Target: `courshare-igw`.
2. **Private Route Table (`courshare-private-rt`):**
   - Assocations: Gắn với `courshare-private-subnet-1` và `2`.
   - Routes: `0.0.0.0/0` -> Target: Network Interface (ENI) của **NAT Instance**.
3. **Database Route Table (`courshare-db-rt`):**
   - Assocations: Gắn với `courshare-db-subnet-1` và `2`.
   - Routes: Chỉ giữ lại route local, không có route ra ngoài Internet để đảm bảo cô lập hoàn toàn cơ sở dữ liệu.

## Bước 6: Khởi tạo Security Groups
Chúng ta sẽ tạo các Security Group (SG) sau:
* **ALB SG (`courshare-alb-sg`):** Cho phép Inbound port `80` (HTTP) và `443` (HTTPS) từ mọi nguồn (`0.0.0.0/0`).
* **ECS Tasks SG (`courshare-ecs-tasks-sg`):** Cho phép Inbound các cổng `8080`, `8081`, `8082`, `8083`, `8084` chỉ từ `courshare-alb-sg`.
* **RDS Database SG (`courshare-rds-sg`):** Cho phép Inbound cổng `5432` (PostgreSQL) chỉ từ `courshare-ecs-tasks-sg` và NAT Instance (làm Bastion Host).

<!-- TODO: chèn screenshot - [Màn hình AWS VPC Console hiển thị danh sách 6 subnets được gán đúng dải CIDR và Availability Zones] -->

<!-- TODO: chèn screenshot - [Màn hình AWS VPC Route Table cấu hình dòng định tuyến 0.0.0.0/0 trỏ về Elastic Network Interface (eni) của NAT Instance] -->

<!-- TODO: chèn screenshot - [Màn hình Inbound Rules của Security Group courshare-rds-sg chỉ nhận traffic port 5432 từ Security Group của ECS Tasks] -->
