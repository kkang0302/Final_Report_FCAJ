---
title : "Chuẩn bị"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

# Chuẩn bị môi trường triển khai

Trước khi bắt đầu các bước triển khai CourShare lên AWS, bạn cần chuẩn bị đầy đủ tài khoản, quyền hạn và công cụ phát triển cần thiết dưới đây.

## 1. Tài khoản AWS & Quyền hạn
* Bạn cần có một **tài khoản AWS** hoạt động (tận dụng tối đa AWS Free Tier để tiết kiệm chi phí).
* Tránh sử dụng tài khoản Root. Hãy tạo một **IAM User** có quyền AdministratorAccess (hoặc các quyền hạn hẹp hơn liên quan đến VPC, ECS, RDS, S3, Cognito, SQS, CloudWatch, ECR).
* Cấu hình bảo mật Multi-Factor Authentication (MFA) cho tài khoản quản trị này.

## 2. Công cụ phát triển trên máy Client
Hãy tải và cài đặt các công cụ sau trên máy tính cá nhân của bạn:

* **AWS CLI (v2):** Giao diện dòng lệnh giúp tương tác với các tài nguyên AWS từ terminal máy local.
  - Tải về tại: [AWS CLI Installation Guide](https://aws.amazon.com/cli/)
  - Cấu hình AWS CLI bằng lệnh:
    ```bash
    aws configure
    ```
    Nhập Access Key ID, Secret Access Key, Region mặc định (ví dụ `us-east-1`) và định dạng output là `json`.
* **Docker Desktop:** Cực kỳ quan trọng để xây dựng, chạy thử nghiệm và đẩy Docker container images của các microservices lên Amazon ECR.
  - Tải về tại: [Docker Desktop](https://www.docker.com/products/docker-desktop/)
* **Git:** Quản lý mã nguồn dự án và kết nối tới GitHub phục vụ dựng CI/CD pipeline.
* **Node.js (v18+) & Java JDK 17:** Dành cho việc chạy thử và phát triển mã nguồn microservices local (Spring Boot và Express Node.js).
* **Một Database Client:** Ví dụ **DBeaver** hoặc **pgAdmin** để kết nối và kiểm tra dữ liệu RDS PostgreSQL.

## 3. Mã nguồn các dịch vụ CourShare
Đảm bảo bạn đã clone đầy đủ 6 repository dự án dưới đây về máy của mình:
* Identity Service (Java Spring Boot)
* Course Service (Java Spring Boot)
* API Gateway (Spring Cloud Gateway)
* Enrollment Service (Node.js/Express)
* Learning Service (Node.js/Express)
* Payment Service (Node.js/Express)

<!-- TODO: chèn screenshot - [Màn hình Terminal chạy lệnh `aws --version` và `docker --version` hiển thị các phiên bản phần mềm đã cài đặt thành công] -->
