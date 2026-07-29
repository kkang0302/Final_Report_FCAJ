---
title : "Xử lý bất đồng bộ"
date : 2024-01-01 
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

# Tách rời hệ thống bằng Xử lý Bất đồng bộ qua Amazon SQS & Workers

Trong hệ thống CourShare, các tác vụ như chuyển đổi định dạng video (transcoding) và gửi email thông báo giao dịch là những công việc tốn tài nguyên và thời gian. Chúng ta tách rời (decouple) các tác vụ này khỏi luồng xử lý API đồng bộ bằng cách sử dụng hàng đợi thông điệp **Amazon SQS** kết hợp với các dịch vụ chạy nền **VideoWorker** và **NotificationWorker**.

## Bước 1: Khởi tạo các Amazon SQS Queues
1. Vào **SQS Console** -> **Create queue**.
2. Tạo 2 hàng đợi dạng **Standard**:
   - `courshare-video-transcode-queue` (Dành cho tác vụ transcoding video).
   - `courshare-notification-queue` (Dành cho các email thông báo giao dịch và đăng ký học).
3. Thiết lập **Default Visibility Timeout** lên 10 phút (600 giây) đối với video transcode queue (Vì tác vụ transcode video nặng cần nhiều thời gian xử lý, tránh trường hợp thông điệp bị đẩy lại hàng đợi khi worker vẫn đang xử lý).

## Bước 2: Dựng và Triển khai VideoWorker (Transcode HLS)
Khi giảng viên upload video gốc (MP4) lên S3 Media bucket thành công, Course Service gửi thông điệp dạng JSON chứa `bucketName` và `objectKey` vào `courshare-video-transcode-queue`.

**VideoWorker** chạy liên tục dưới dạng ECS Service trong Private Subnet thực hiện:
1. Thăm dò (Long Polling) tin nhắn từ hàng đợi SQS thông qua AWS SDK:
   ```javascript
   const { SQSClient, ReceiveMessageCommand, DeleteMessageCommand } = require("@aws-sdk/client-sqs");
   // Long polling bằng cách đặt WaitTimeSeconds = 20
   ```
2. Khi nhận được tin nhắn, VideoWorker tải video gốc `.mp4` từ S3 Media bucket về ổ đĩa cục bộ của task.
3. Thực thi thư viện **ffmpeg** thực hiện chuyển đổi định dạng sang HLS (HTTP Live Streaming):
   ```bash
   ffmpeg -i input.mp4 -profile:v baseline -level 3.0 -s 640x360 -start_number 0 -hls_time 10 -hls_list_size 0 -f hls index.m3u8
   ```
   *Quá trình này sinh ra một file chỉ mục `index.m3u8` và hàng loạt tệp phân đoạn video nhỏ `.ts` (ví dụ `index0.ts`, `index1.ts`...).*
4. Upload toàn bộ các file `.m3u8` và `.ts` lên thư mục bài học tương ứng trong S3 Media Bucket.
5. Xóa tin nhắn khỏi hàng đợi SQS để xác nhận xử lý thành công.

## Bước 3: Triển khai NotificationWorker (Gửi email thông báo)
Tương tự, khi xảy ra các giao dịch thanh toán hoặc đăng ký học thành công, các microservices tương ứng bắn thông điệp chứa thông tin liên hệ và nội dung thông báo vào `courshare-notification-queue`.

**NotificationWorker** lắng nghe queue này, đọc thông tin và gọi dịch vụ gửi email để gửi thông báo trực tiếp tới hòm thư của học viên và giảng viên, giúp giảm tải tối đa độ trễ phản hồi API cho người dùng.

<!-- TODO: chèn screenshot - [Màn hình AWS SQS Console hiển thị danh sách các Queue kèm URL và thuộc tính Visibility Timeout được cấu hình chính xác] -->

<!-- TODO: chèn screenshot - [Màn hình S3 Media Bucket hiển thị thư mục khóa học chứa file chỉ mục index.m3u8 và các tệp phân đoạn ts đã được VideoWorker transcode và upload thành công] -->
