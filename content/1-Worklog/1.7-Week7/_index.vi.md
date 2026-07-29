---
title: "Worklog Tuần 7"
date: 2026-07-27
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

* Thực hiện kiểm thử tích hợp (Integration Testing) toàn diện hệ thống từ đầu cuối (End-to-End).
* Xác minh kịch bản kiểm soát truy cập (Access Control) qua Lambda Authorizer.
* Tối ưu hóa hiệu năng cơ sở dữ liệu PostgreSQL trên Amazon RDS.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Thiết lập kịch bản kiểm thử E2E: Đăng ký/Đăng nhập -> Xem khóa học -> Mua khóa học qua Stripe -> Ghi danh -> Học bài và đánh dấu tiến trình | 27/07/2026 | 27/07/2026 | E2E Testing Scenarios |
| 3 | - Kiểm thử tính năng phân quyền: xác thực lỗi JWT, chặn người dùng thường truy cập API quản trị của giảng viên/admin | 28/07/2026 | 28/07/2026 | RBAC Security Specifications |
| 4 | - Tối ưu hóa DB: Tạo các chỉ mục (indexes) cho cột thường xuyên tra cứu như `email` (bảng users), `course_id` (bảng lessons) | 29/07/2026 | 29/07/2026 | PostgreSQL Optimization Guides |
| 5 | - Rà soát SQL queries tự động sinh ra bởi Prisma ORM và Spring Data JPA nhằm phát hiện và sửa lỗi truy vấn $N+1$ | 30/07/2026 | 30/07/2026 | Database N+1 Query Auditing |
| 6 | - Cấu hình Connection Pool (HikariCP / Prisma Connection Limits) kết nối RDS PostgreSQL để kiểm soát giới hạn kết nối đồng thời | 31/07/2026 | 31/07/2026 | RDS Connection Pool Configuration |

### Kết quả đạt được tuần 7:

* Kiểm thử thành công toàn bộ luồng nghiệp vụ liên thông (E2E) từ khi đăng ký tài khoản cho đến lúc xem và hoàn thành khóa học.
* Lambda Authorizer và phân quyền RBAC (Role-Based Access Control) hoạt động chính xác, đảm bảo dữ liệu an toàn.
* Tốc độ truy vấn cơ sở dữ liệu cải thiện rõ rệt, lỗi truy vấn dư thừa được khắc phục triệt để nhờ tối ưu index và tối giản các truy vấn $N+1$.
* Connection pool hoạt động tối ưu, đảm bảo hệ thống không bị quá tải kết nối khi lưu lượng truy cập cao.
* **Xác minh môi trường ECS Fargate & Cloud Map**: Xác nhận các container microservices được đẩy lên Amazon ECR và chạy ổn định trên Amazon ECS; kiểm tra cơ chế phân giải tên miền động qua Cloud Map (Service Discovery) giúp các service kết nối an toàn trong mạng nội bộ.

![Amazon ECR](/images/1-WorkLog/ECR.png)
*Danh sách các repositories lưu trữ Docker images trên Amazon Elastic Container Registry (ECR).*

![Amazon ECS Cluster](/images/1-WorkLog/ECS.png)
*Giao diện điều khiển Amazon ECS hiển thị Cluster của hệ thống.*

![Amazon ECS Services](/images/1-WorkLog/ECS-2.png)
*Các dịch vụ chạy dưới dạng ECS Tasks hoạt động ổn định ở trạng thái ACTIVE.*

![Amazon Cloud Map Namespaces](/images/1-WorkLog/Namespaces.png)
*Namespace nội bộ `courshare.local` được quản lý bởi Amazon Cloud Map cho việc định vị dịch vụ (Service Discovery).*

