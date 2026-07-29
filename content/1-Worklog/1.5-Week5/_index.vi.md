---
title: "Worklog Tuần 5"
date: 2026-07-13
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

* Triển khai hệ thống cổng định tuyến API Gateway tập trung cho dự án CourShare.
* Xây dựng cơ chế xác thực người dùng tập trung bằng AWS Lambda Custom Authorizer sử dụng chữ ký số JWT.
* Liên kết API Gateway với mạng Private thông qua VPC Link và Internal Application Load Balancer.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Xây dựng AWS Lambda Custom Authorizer bằng Node.js để giải mã JWT, xác thực chữ ký và trích xuất thông tin người dùng | 13/07/2026 | 13/07/2026 | AWS Lambda Authorizer SDK |
| 3 | - Cấu hình HTTP API Gateway làm điểm tiếp nhận duy nhất cho toàn bộ request từ Client và thiết lập chính sách CORS | 14/07/2026 | 14/07/2026 | HTTP API Gateway Setup |
| 4 | - Thiết lập VPC Link liên kết API Gateway với Internal Application Load Balancer (ALB) trong mạng Private | 15/07/2026 | 15/07/2026 | VPC Link Integration |
| 5 | - Cấu hình Parameter Mapping tại API Gateway để inject thông tin user từ Lambda Authorizer vào Header của request tuần tự | 16/07/2026 | 16/07/2026 | API Gateway Parameter Mapping Guide |
| 6 | - Thiết lập Listener Rules trên Internal ALB để định hướng traffic (đường dẫn `/auth*`, `/courses*`, `/checkout*`, `/enrollments*`, `/progress*`...) về các Target Groups tương ứng | 17/07/2026 | 17/07/2026 | ALB Routing Rules Guide |

### Kết quả đạt được tuần 5:

* Toàn bộ các request đi qua API Gateway đều được xác thực và phân quyền tập trung bằng Lambda Authorizer một cách an toàn.
* API Gateway chuyển tiếp thông tin định danh (`X-User-Id`, `X-User-Email`, `X-User-Roles`) xuống backend trong Private Subnets thông qua VPC Link và Internal ALB.
* Hệ thống phân luồng định tuyến (Path-based Routing) hoạt động chính xác đến từng microservices thành phần.

![ALB Rules](/images/1-WorkLog/ALB.png)
*Các quy tắc định tuyến (Listener Rules) trên Application Load Balancer để chuyển tiếp traffic theo đường dẫn URL.*

