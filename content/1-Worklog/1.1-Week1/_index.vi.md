---
title: "Worklog Tuần 1"
date: 2026-06-15
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu tuần 1:

* Nghiên cứu mô hình phát triển ứng dụng Cloud-native và so sánh kiến trúc Monolithic với Microservices.
* Thiết kế sơ đồ kiến trúc hệ thống tổng quan cho dự án CourShare.
* Phân tích và xác định vai trò của các dịch vụ đám mây AWS cốt lõi trong sơ đồ vận hành.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 1 | - Nghiên cứu và so sánh kiến trúc Monolithic truyền thống với Microservices chạy trên môi trường Cloud <br> - Đánh giá khả năng mở rộng độc lập, tính sẵn sàng cao và khả năng cô lập lỗi | 15/06/2026 | 15/06/2026 | Cloud-native patterns & Microservices guide |
| 2 | - Xác định và thiết kế hệ thống CourShare gồm 5 dịch vụ thành phần (Identity, Course, Payment, Enrollment, Learning Services) | 16/06/2026 | 16/06/2026 | Tài liệu thiết kế hệ thống CourShare |
| 3 | - Thiết kế luồng đi dữ liệu qua API Gateway (Single Entry Point) đi qua cơ chế phân quyền tập trung tại Lambda Authorizer | 17/06/2026 | 17/06/2026 | AWS API Gateway & Lambda Authorizer Documentation |
| 4 | - Thiết kế luồng chuyển tiếp yêu cầu từ API Gateway đến mạng riêng (Private Subnets) thông qua Internal Application Load Balancer (ALB) | 18/06/2026 | 18/06/2026 | AWS ALB Developer Guide |
| 5 | - Phác thảo sơ đồ thực thể liên kết (ERD) ban đầu cho các cơ sở dữ liệu của microservices | 19/06/2026 | 19/06/2026 | Database Design & ERD Best Practices |

### Kết quả đạt được tuần 1:

* Hoàn thành bản vẽ kiến trúc hệ thống tổng thể và phân rã chức năng thành 5 microservices chạy trên các cổng riêng biệt (8081, 8082, 8083, 8084, 8085).
* Định rõ luồng bảo mật tập trung qua Custom Authorizer trước khi chuyển traffic vào mạng riêng.
* Phác thảo sơ đồ thực thể liên kết (ERD) ban đầu cho các microservices, định hình rõ vai trò của từng dịch vụ đám mây sẽ sử dụng.
* **Cơ sở hạ tầng tài khoản ban đầu**: Khởi tạo thành công tài khoản AWS, thiết lập tài khoản IAM Admin User có cấu hình MFA bảo mật và cài đặt AWS CLI trên máy cục bộ để kết nối và kiểm tra quyền truy cập.

![IAM Console](/images/1-WorkLog/iam-console.png)
*Màn hình AWS IAM Console hiển thị tài khoản quản trị Admin đã cấu hình bảo mật MFA.*

![AWS CLI Terminal](/images/1-WorkLog/terminal.png)
*Kiểm tra kết nối AWS CLI thành công bằng lệnh `aws sts get-caller-identity` trên máy cá nhân.*

