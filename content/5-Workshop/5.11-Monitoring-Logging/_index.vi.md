---
title : "Giám sát & Ghi log"
date : 2024-01-01 
weight : 11
chapter : false
pre : " <b> 5.11. </b> "
---

# Giám sát & Ghi log tập trung với Amazon CloudWatch

Trong kiến trúc Microservices phân tán, việc theo dõi trạng thái hoạt động và phân tích lỗi gặp nhiều khó khăn nếu logs của từng container bị phân mảnh. Chúng ta thiết lập cơ chế ghi log tập trung (Centralized Logging) toàn bộ container stdout về **Amazon CloudWatch Logs**, giúp dễ dàng giám sát lỗi thời gian thực từ một giao diện duy nhất.

## 1. Cơ chế hoạt động của Log Driver `awslogs`
AWS ECS Fargate hỗ trợ driver ghi log mặc định là `awslogs`. Driver này tự động bắt toàn bộ các dòng log in ra màn hình console (Standard Output/Standard Error) của ứng dụng bên trong container và chuyển hướng chúng về CloudWatch Logs.

## 2. Cấu hình gửi log trong Task Definition
Trong tệp cấu hình Task Definition của mỗi microservice, phần cấu hình container chứa khai báo ghi log:

```json
"logConfiguration": {
    "logDriver": "awslogs",
    "options": {
        "awslogs-group": "/ecs/courshare-microservices",
        "awslogs-region": "us-east-1",
        "awslogs-stream-prefix": "identity-service",
        "awslogs-create-group": "true"
    }
}
```

* **awslogs-group:** Tên nhóm log chung cho hệ thống, ví dụ `/ecs/courshare-microservices`.
* **awslogs-stream-prefix:** Tiền tố của log stream để phân biệt nguồn log từ các service khác nhau (ví dụ: `identity-service`, `course-service`, `video-worker`...).

## 3. Xem logs trên CloudWatch Console
1. Truy cập **CloudWatch Console** -> **Logs** -> **Log groups**.
2. Nhấp chọn log group `/ecs/courshare-microservices`.
3. Bạn sẽ nhìn thấy danh sách các log streams tương ứng với từng container task đang hoạt động.
4. Sử dụng tính năng **CloudWatch Logs Insights** để chạy các câu truy vấn tìm kiếm log lỗi nhanh chóng trên tất cả các dịch vụ:
   ```sql
   fields @timestamp, @message
   | filter @message like /Error/ or @message like /Exception/
   | sort @timestamp desc
   | limit 100
   ```

<!-- TODO: chèn screenshot - [Màn hình Amazon CloudWatch Logs Groups hiển thị nhóm log /ecs/courshare-microservices] -->

<!-- TODO: chèn screenshot - [Màn hình CloudWatch Logs Insights đang chạy câu lệnh truy vấn lọc các log chứa lỗi Error từ container] -->
