---
title: "Event 1"
date: 2026-06-20
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Báo cáo tóm tắt: "Cloud Architecture Battle 1"

### Mục tiêu sự kiện

- Củng cố và kiểm chứng các khái niệm cốt lõi về kiến trúc đám mây dưới áp lực thi đấu.
- Thử nghiệm tốc độ và độ chính xác khi giải quyết các tình huống thiết kế hệ thống.
- Thúc đẩy tinh thần đưa ra quyết định đồng thuận và hợp tác trong đội nhóm.

### Thể lệ và hình thức thi đấu

Sự kiện được tổ chức dưới dạng một giải đấu tính điểm đầy kịch tính:
1. Hai đội trực tiếp đối đầu nhau.
2. Các đội cùng trả lời một chuỗi câu hỏi trắc nghiệm (multiple choice) về kiến trúc đám mây.
3. Độ khó của câu hỏi tăng dần, từ các kiến thức nền tảng đến các mô hình thiết kế hệ thống phức tạp hơn.
4. Điểm số được tính dựa trên độ chính xác và tốc độ phản hồi để xếp hạng đội thắng cuộc sau các lượt đấu.

### Kiến thức Cloud Architecture tích lũy

#### 1. Thiết kế ứng dụng đa tầng & Khả năng mở rộng
- Hiểu rõ cách phân bổ mạng con công khai (public subnet) và mạng con riêng tư (private subnet) để cô lập tài nguyên an toàn.
- Củng cố cấu hình bộ cân bằng tải Application Load Balancer (ALB) và Auto Scaling Groups (ASG) để tự động mở rộng tài nguyên dựa trên mức sử dụng CPU hoặc số lượng yêu cầu.
- Nắm vững các mô hình triển khai Multi-AZ cho Amazon RDS nhằm đảm bảo tính sẵn sàng cao cho cơ sở dữ liệu.

#### 2. Chiến lược bộ nhớ đệm (Caching) & Phân phối tải đọc dữ liệu
- Xác định thời điểm tích hợp Amazon ElastiCache (Redis/Memcached) để lưu trữ trạng thái phiên làm việc (session state) và bộ nhớ đệm cho các truy vấn database thường gặp.
- Phân biệt rõ ràng vai trò của Read Replicas (để mở rộng khả năng đọc dữ liệu) so với triển khai Multi-AZ (phục vụ khắc phục thảm họa và duy trì tính sẵn sàng).

#### 3. Xử lý bất đồng bộ & Khử liên kết hệ thống (Decoupling)
- Sử dụng hàng đợi thông điệp như Amazon SQS để làm đệm điều tiết khi lưu lượng truy cập của người dùng tăng đột biến, tránh làm quá tải ứng dụng phía sau.
- Kết hợp thông báo sự kiện từ Amazon S3 với AWS Lambda để xử lý tệp tin tự động theo mô hình không máy chủ (serverless), giúp tối ưu hóa chi phí.

### Hình ảnh sự kiện

![Thuyết trình giải pháp kiến trúc](/images/4-EventParticipated/4.1-Event1/20260711_085511.jpg)

![Thảo luận thiết kế kiến trúc nhóm](/images/4-EventParticipated/4.1-Event1/Messenger_creation_FBFEF47E-328E-4FB3-9A31-092EEC44854A.png)
