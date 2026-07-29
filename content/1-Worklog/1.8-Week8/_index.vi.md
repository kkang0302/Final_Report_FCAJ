---
title: "Worklog Tuần 8"
date: 2026-08-03
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:

* Thiết lập hệ thống ghi log tập trung và giám sát tài nguyên thông qua Amazon CloudWatch.
* Dọn dẹp tài nguyên cấu hình, chuẩn bị tệp tin mã nguồn bàn giao.
* Hoàn thiện slide báo cáo, báo cáo thực tập chi tiết và chuẩn bị demo sản phẩm.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Cấu hình gửi log tập trung từ các ECS tasks chạy microservices và log thực thi của Lambda Authorizer về Amazon CloudWatch Logs | 03/08/2026 | 03/08/2026 | CloudWatch Logs Integration |
| 3 | - Giám sát và thiết lập các cảnh báo vượt ngưỡng tài nguyên phần cứng (CPU, Memory) cho RDS PostgreSQL và NAT Instance | 04/08/2026 | 04/08/2026 | CloudWatch Metrics & Alarms Guide |
| 4 | - Dọn dẹp, tổng hợp mã nguồn dự án và hoàn thiện file nén hạ tầng Terraform (`terraform.zip`) phục vụ bàn giao | 05/08/2026 | 05/08/2026 | Project Handover Procedures |
| 5 | - Soạn thảo báo cáo thực tập chương 2, vẽ lại sơ đồ hạ tầng thực tế và sơ đồ luồng dữ liệu của CourShare | 06/08/2026 | 06/08/2026 | Internship Report Structure Templates |
| 6 | - Thiết lập slide thuyết trình tổng kết 8 tuần thực tập và chạy thử demo sản phẩm trước hội đồng đánh giá | 07/08/2026 | 07/08/2026 | Final Evaluation Requirements |

### Kết quả đạt được tuần 8:

* Hệ thống CourShare vận hành an toàn với đầy đủ cơ chế Logging và Monitoring tập trung trên CloudWatch.
* Báo cáo thực tập được hoàn thiện chi tiết cả về mặt lý thuyết mạng và quy trình kỹ thuật triển khai dự án thực tế.
* Chuẩn bị đầy đủ tài nguyên bàn giao (mã nguồn, Terraform file) và slide thuyết trình sẵn sàng cho buổi bảo vệ kết quả thực tập.

<!-- TODO: chèn screenshot - [Giao diện CloudWatch Logs hiển thị danh sách các Log Groups của ECS Services] -->
<!-- TODO: chèn screenshot - [Biểu đồ giám sát CPU/Memory Utilization của RDS PostgreSQL trên CloudWatch Metrics] -->