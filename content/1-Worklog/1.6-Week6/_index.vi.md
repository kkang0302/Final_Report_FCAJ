---
title: "Worklog Tuần 6"
date: 2026-07-20
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:

* Thiết lập hệ thống lưu trữ tĩnh và phương tiện video bài học trên Amazon S3.
* Triển khai mô hình Event-driven tích hợp S3 Event Notification với hàng đợi tin nhắn Amazon SQS.
* Xây dựng module worker để xử lý tác vụ media (transcoding HLS) bất đồng bộ từ hàng đợi.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Khởi tạo các S3 Buckets: `courshare-frontend-web-bucket` và `courshare-media-hls-bucket` để quản lý asset và video | 20/07/2026 | 20/07/2026 | Amazon S3 Storage Classes & Setup |
| 3 | - Cấu hình S3 Event Notification gửi tin nhắn đến hàng đợi Amazon SQS (`courshare-video-processing-queue`) khi có video `.mp4` mới | 21/07/2026 | 21/07/2026 | S3 Event Notifications Guide |
| 4 | - Thiết lập SQS Queue Policy cho phép S3 thực hiện hành động `sqs:SendMessage` với điều kiện ràng buộc ARN nguồn | 22/07/2026 | 22/07/2026 | Amazon SQS Security Policies |
| 5 | - Phát triển worker Node.js thực hiện polling các tin nhắn từ SQS chứa thông tin file media mới tải lên | 23/07/2026 | 23/07/2026 | Node.js AWS SDK for SQS |
| 6 | - Kiểm thử tích hợp: tải lên video và theo dõi cơ chế đẩy thông báo sự kiện sang hàng đợi SQS | 24/07/2026 | 24/07/2026 | Event-driven integration testing |

### Kết quả đạt được tuần 6:

* Phân vùng lưu trữ Amazon S3 được khởi tạo và phân quyền bảo mật chặt chẽ.
* Thiết lập thành công chuỗi thông báo tự động (Event-driven) giữa S3 và SQS để xử lý video không đồng bộ.
* SQS Queue Policy được cấu hình an toàn, chặn các nguồn tin gửi tin giả mạo.
* Worker Node.js chạy ngầm bắt được các thông báo tải video lên hàng đợi để sẵn sàng cho các tác vụ chuyển đổi video (Transcoding) sau này.

![S3 Buckets](/images/1-WorkLog/Bucket.png)
*Danh sách các Amazon S3 Buckets được khởi tạo để lưu trữ mã nguồn frontend tĩnh và media.*

![CloudFront Distribution](/images/1-WorkLog/Distribution.png)
*Cấu hình các phân phối CloudFront Distributions cho hệ thống CourShare.*

