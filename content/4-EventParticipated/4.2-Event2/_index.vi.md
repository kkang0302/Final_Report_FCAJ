---
title: "Event 2"
date: 2026-07-11
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# Báo cáo tóm tắt: "Cloud Architecture Battle 2"

### Mục tiêu sự kiện

- Làm chủ các khái niệm kiến trúc doanh nghiệp nâng cao, bao gồm khắc phục thảm họa (DR), kết nối mạng đám mây lai (hybrid cloud) và bảo mật thông tin.
- Thực hành kỹ năng ra quyết định nhanh chóng trong thiết kế hệ thống dưới áp lực thi đấu đối kháng trực tiếp.
- Củng cố kiến thức lý thuyết về các thực hành tốt nhất trên AWS theo bộ khung Well-Architected Framework.

### Thể lệ và hình thức thi đấu

Tiếp tục hình thức thi đấu từ Battle 1, giải đấu nâng cấp mức độ thách thức:
1. Hai đội bước vào vòng tranh tài quyết định.
2. Các đội trả lời các câu hỏi trắc nghiệm phức tạp xoay quanh các kịch bản lỗi hệ thống thực tế, các phương án di trú hạ tầng (migration) và các tiêu chuẩn bảo mật.
3. Các câu hỏi giả lập các sự cố sản xuất thực tế, yêu cầu tư duy phân tích sâu sắc để tìm ra phương án tối ưu.
4. Điểm số chung cuộc từ các câu hỏi này quyết định nhà vô địch của giải đấu.

### Kiến thức Cloud Architecture tích lũy

#### 1. Chiến lược Khắc phục thảm họa (Disaster Recovery) ở quy mô lớn
- Tìm hiểu chuyên sâu cách xác định và tối ưu các chỉ số RPO (Recovery Point Objective) và RTO (Recovery Time Objective).
- So sánh, đánh giá các cơ chế DR khác nhau trên AWS: Sao lưu & Khôi phục (Backup & Restore), Pilot Light, Warm Standby và Multi-Site Active-Active.
- Ứng dụng Aurora Global Database để đồng bộ hóa cơ sở dữ liệu liên vùng nhanh chóng cùng dịch vụ Route 53 Application Recovery Controller giúp tự động chuyển vùng khi có thảm họa xảy ra.

#### 2. Kết nối mạng đám mây lai (Hybrid Cloud Network) cho doanh nghiệp lớn
- Thiết kế các đường truyền mạng riêng tư và bảo mật giữa trung tâm dữ liệu on-premises của doanh nghiệp và đám mây AWS.
- Phân bổ hạ tầng và lựa chọn các dịch vụ AWS Direct Connect (DX), AWS Transit Gateway, và các kênh truyền mã hóa IPSec VPN để tối ưu chi phí, hiệu năng cũng như tính dự phòng cho hệ thống.

#### 3. Bảo mật nâng cao & Tuân thủ tiêu chuẩn doanh nghiệp
- Thực thi bảo vệ dữ liệu bằng AWS Key Management Service (KMS) kết hợp với các khóa do khách hàng tự quản lý (Customer Managed Keys - CMK) cùng chính sách phân quyền khóa (key policy) chặt chẽ.
- Bảo vệ các ứng dụng web khỏi các lỗ hổng bảo mật phổ biến và tấn công từ chối dịch vụ (DDoS) thông qua việc kết hợp AWS WAF, AWS Shield và Amazon CloudFront.
- Thiết kế hệ thống tuân thủ các tiêu chuẩn bảo mật nghiêm ngặt như PCI-DSS và HIPAA thông qua AWS Artifact và các thực hành bảo mật chuẩn hóa.

### Phần chia sẻ từ các Diễn giả khách mời

Vào cuối buổi sự kiện, chúng tôi đã có cơ hội lắng nghe những chia sẻ thực chiến cực kỳ bổ ích từ hai chuyên gia trong ngành:

#### 1. Chia sẻ từ Kỹ sư DevSecOps: AWS Security Agent
- **Nội dung chính:** Bảo mật các máy chủ ảo EC2 và container runtimes.
- **Kiến thức cốt lõi:** Hướng dẫn triển khai và quản trị các tác nhân bảo mật như **AWS Systems Manager (SSM) Agent** và **Amazon Inspector Agent** để liên tục quét lỗ hổng bảo mật, kiểm tra việc tuân thủ các bản vá (patch compliance), và phát hiện mối đe dọa trong thời gian thực. Nhấn mạnh việc tích hợp các tác nhân này vào đường ống CI/CD để tự động hóa quét bảo mật.

#### 2. Chia sẻ từ Kỹ sư NOC: Thỏa thuận mức dịch vụ (SLA)
- **Nội dung chính:** Vận hành thực tế, giám sát thời gian hoạt động (uptime) và cách tính toán độ sẵn sàng của hệ thống.
- **Kiến thức cốt lõi:** Phân tích sâu cách tính toán SLA (ví dụ: sự khác biệt giữa độ sẵn sàng 99.9% và 99.99%) cùng các tác động về mặt tài chính lẫn pháp lý cho doanh nghiệp. Tìm hiểu cách đội ngũ NOC giám sát hệ thống thời gian thực nhằm phát hiện và xử lý sớm các bất thường trước khi vi phạm cam kết SLA, sử dụng Amazon CloudWatch và Service Health dashboard.

### Hình ảnh sự kiện

![Thảo luận thiết kế kiến trúc nâng cao](/images/4-EventParticipated/4.2-Event2/Messenger_creation_B6B2B4BE-ECB8-4DE6-B98E-F8461010D4B1.png)

