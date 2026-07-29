---
title: "Worklog Tuần 2"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

* Thiết lập hệ thống cơ sở dữ liệu đồng nhất cho các microservices.
* Phát triển dịch vụ xác thực và phân quyền (Identity Service) bằng Java Spring Boot.
* Phát triển dịch vụ quản lý nội dung khóa học (Course Service) bằng Java Spring Boot.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Thiết kế cấu trúc bảng dữ liệu cho Identity Service và Course Service <br> - Định nghĩa schema cơ sở dữ liệu sử dụng Prisma ORM và khởi tạo các phiên bản migration | 22/06/2026 | 22/06/2026 | Prisma ORM Schema Reference |
| 3 | - Phát triển chức năng đăng ký, đăng nhập (Access/Refresh Token) và đăng xuất (Blacklist tokens qua Redis) trên Identity Service | 23/06/2026 | 23/06/2026 | Spring Security & JWT Guidelines |
| 4 | - Tích hợp BCrypt băm mật khẩu và cấu hình tự động khởi tạo các vai trò mặc định (`STUDENT`, `INSTRUCTOR`, `ADMIN`) | 24/06/2026 | 24/06/2026 | Spring Boot Security Config |
| 5 | - Xây dựng REST API quản lý vòng đời của khóa học trên Course Service (Danh mục, thông tin chi tiết, chương học và bài học) | 25/06/2026 | 25/06/2026 | Course Management API Docs |
| 6 | - Kiểm thử cục bộ các API xác thực người dùng và quản lý khóa học dưới môi trường local | 26/06/2026 | 26/06/2026 | Postman API Testing Guide |

### Kết quả đạt được tuần 2:

* Hoàn thành thiết kế schema DB chi tiết, quản lý migration nhất quán thông qua Prisma ORM.
* Hoàn thiện Identity Service với đầy đủ chức năng Authentication & Authorization bảo mật cao thông qua JWT và Redis Blacklist.
* Hoàn thiện REST API cho Course Service đáp ứng các yêu cầu quản lý danh mục và bài giảng.
* Kiểm thử cục bộ (Local Testing) thành công toàn bộ các endpoints chính của 2 service trên.
* **Đóng gói Container ban đầu**: Viết Dockerfile tối ưu và chạy thử container cho các dịch vụ cốt lõi dưới môi trường Docker cục bộ để đảm bảo tính độc lập môi trường trước khi đưa lên đám mây.

![Docker Image List](/images/1-WorkLog/docker-identity-service-2.png)
*Danh sách các Docker Image đã được build cục bộ phục vụ cho các microservices.*

![Docker Container Running](/images/1-WorkLog/docker-identity-service.png)
*Khởi chạy container của Identity Service trên môi trường Docker Desktop cục bộ tại cổng 8080.*