---
title: "Worklog Tuần 3"
date: 2026-06-29
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

* Phát triển 3 microservices còn lại (Payment, Enrollment, Learning Services) sử dụng Node.js/Express và Prisma ORM.
* Tích hợp cổng thanh toán bên thứ ba Stripe để xử lý giao dịch mua khóa học.
* Viết unit tests bằng Jest để đảm bảo tính ổn định và chính xác của các API.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Phát triển Payment Service: tích hợp Stripe SDK để khởi tạo Checkout Session phục vụ học viên thanh toán trực tuyến | 29/06/2026 | 29/06/2026 | Stripe API Reference |
| 3 | - Thiết lập REST API endpoint lắng nghe sự kiện Stripe Webhook (`checkout.session.completed`) để cập nhật trạng thái thanh toán và ghi nhận giao dịch | 30/06/2026 | 30/06/2026 | Stripe Webhook Developers Guide |
| 4 | - Phát triển Enrollment Service: Xây dựng API xác thực quyền sở hữu/đăng ký khóa học và lưu trữ thông tin ghi danh sau khi mua thành công | 01/07/2026 | 01/07/2026 | Enrollment System Logic |
| 5 | - Phát triển Learning Service: Xây dựng API đánh dấu bài học hoàn thành, tính toán phần trăm tiến trình học tập của học viên | 02/07/2026 | 02/07/2026 | Learning Progress Specifications |
| 6 | - Viết và chạy các Unit Test bằng framework Jest cho cả 3 microservices Node.js trước khi đóng gói Docker | 03/07/2026 | 03/07/2026 | Jest Testing Framework Documentation |

### Kết quả đạt được tuần 3:

* Hoàn thiện mã nguồn cho cả 5 microservices, kiểm thử cục bộ tích hợp hoạt động mượt mà.
* Tích hợp thành công cổng thanh toán Stripe và kiểm thử webhook thành công bằng công cụ Stripe CLI.
* Các API của Payment, Enrollment và Learning Service được xác thực chất lượng thông qua bộ unit test của Jest.

<!-- TODO: chèn screenshot - [Màn hình Stripe Webhook Dashboard local ghi nhận log event checkout.session.completed status 200] -->
<!-- TODO: chèn screenshot - [Kết quả chạy Jest test suite thành công trên terminal của các service Node.js] -->
