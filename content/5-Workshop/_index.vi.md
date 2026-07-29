---
title: "Workshop"
date: 2026-07-28
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Triển khai Nền tảng CourShare trên Điện toán Đám mây AWS

Trong chương này, chúng ta sẽ cùng thực hành từng bước thiết lập và triển khai toàn bộ hệ thống CourShare lên hạ tầng AWS. Các bước được sắp xếp theo thứ tự logic, từ thiết lập mạng nền tảng, tạo dựng các microservices, thiết lập cơ sở dữ liệu, phân quyền, lưu trữ, truyền thông điệp bất đồng bộ cho tới tự động hóa pipeline triển khai và dọn dẹp tài nguyên.

#### Nội dung bài thực hành

1. [Giới thiệu tổng quan và Kiến trúc hệ thống](5.1-Workshop-overview/)
2. [Chuẩn bị môi trường triển khai](5.2-Prerequisite/)
3. [Thiết lập hạ tầng mạng VPC](5.3-Networking-setup/)
4. [Triển khai Microservices với AWS ECS Fargate & ALB](5.4-Deploying-ECS-Fargate/)
5. [Triển khai lớp Cơ sở dữ liệu Amazon RDS](5.5-Database-layer/)
6. [Xác thực và Quản lý người dùng với Amazon Cognito](5.6-Authentication-Cognito/)
7. [Tích hợp S3, CloudFront CDN & Upload video bảo mật](5.7-Storage-CDN-Video-Upload/)
8. [Tách rời hệ thống bằng Xử lý Bất đồng bộ qua SQS & Workers](5.8-Asynchronous-Processing/)
9. [Triển khai Payment Service & Ví điện tử nội bộ tích hợp Stripe](5.9-Payment-Service-Wallet/)
10. [Tự động hóa Triển khai với Pipeline CI/CD qua GitHub Actions](5.10-CI-CD-Pipeline/)
11. [Giám sát & Ghi log tập trung với Amazon CloudWatch](5.11-Monitoring-Logging/)
12. [Hướng dẫn Dọn dẹp Tài nguyên tránh phát sinh chi phí](5.12-Cleanup/)