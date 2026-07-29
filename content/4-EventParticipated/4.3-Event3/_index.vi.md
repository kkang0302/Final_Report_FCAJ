---
title: "Event 3"
date: 2026-07-25
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

{{% notice warning %}}
⚠️ **Lưu ý:** Các thông tin dưới đây chỉ nhằm mục đích tham khảo, vui lòng **không sao chép nguyên văn** cho bài báo cáo của bạn kể cả warning này.
{{% /notice %}}

# Báo cáo tóm tắt: "FCAJ x Agentic AI Build Week"

### Mục tiêu sự kiện

- Trình bày và thuyết trình (pitching) các dự án AI Agent được phát triển trong tuần lễ Agentic AI Build Week.
- Chia sẻ thiết kế kiến trúc, chi tiết triển khai và giá trị thực tiễn của các hệ thống AI xây dựng trên nền tảng AWS.
- Học hỏi từ các dự án của bạn bè và nhận phản hồi từ các Solution Architect của AWS cùng ban giám khảo.

### Các điểm nhấn chính

#### Nhóm 1: KFC Ordering Agent (Giải Nhất AWS Track)
- **Ý tưởng cốt lõi:** AI Agent hỗ trợ đặt gà KFC trực tiếp qua các kênh chat như Zalo/WhatsApp mà không cần cài ứng dụng.
- **Bài học quan trọng:** Luôn có bước xác nhận (verify) trước khi gửi đơn hàng để giảm thiểu ảo tưởng (hallucination) của AI; thiết kế kiến trúc dạng mô-đun (Channel Adapter, Agent Core, Memory, Tools) để dễ dàng mở rộng cho các thương hiệu khác.
- **Ý thức về chi phí:** Phân tích kỹ chi phí Bedrock và hạ tầng AWS để chứng minh tính khả thi khi đưa vào vận hành thực tế.

#### Nhóm Signal Scout: Strategic Intelligence Platform (Giải Nhì)
- **Ý tưởng cốt lõi:** Hệ thống Multi-Agent hỗ trợ thu thập thông tin đối thủ, phân tích chiến lược và dự đoán ROI cho doanh nghiệp.
- **Bài học quan trọng:** "Công nghệ không thắng được nghiệp vụ" — tập trung giải quyết đúng pain point thay vì cố gắng phô diễn công nghệ AWS.
- **Tư duy MVP:** Trong các cuộc thi phát triển nhanh, một phiên bản MVP hoàn thành 20% nhưng hoạt động trơn tru tốt hơn nhiều so với một kế hoạch 100% nhưng không thể chạy demo.

#### Nhóm BL: AI Solution Architect Assistant
- **Ý tưởng cốt lõi:** Trợ lý AI hỗ trợ Solution Architect sinh sơ đồ kiến trúc, ước tính chi phí và tạo mẫu IaC (Terraform/CloudFormation).
- **Bài học quan trọng:** AI đóng vai trò tăng năng suất (giảm thời gian chuẩn bị từ 2 ngày xuống 30 phút) chứ không thay thế con người; việc cung cấp ngữ cảnh (tài liệu tiêu chuẩn, chính sách công ty) quyết định chất lượng đầu ra của AI.

### Bài học kinh nghiệm tích lũy

- **Tập trung vào vấn đề (Problem First):** Luôn bắt đầu từ các bài toán thực tế của người dùng, không xuất phát từ việc muốn sử dụng công nghệ hay dịch vụ AWS nào.
- **Nghệ thuật Storytelling:** Bắt đầu bài thuyết trình bằng các case study hoặc câu chuyện thực tế để người nghe dễ dàng thấu hiểu pain point.
- **Tư duy Kinh doanh & Thực thi:** Một bản demo chạy được giúp giải quyết trực tiếp nhu cầu thực tế luôn có giá trị hơn một kiến trúc phức tạp trên giấy.
- **Làm việc nhóm:** Sẵn sàng tranh luận cởi mở nhưng cần nhanh chóng thống nhất và cùng nhau thực hiện theo một hướng đi chung.

### Áp dụng vào công việc

- Chuyển dịch tư duy tập trung vào việc làm rõ yêu cầu nghiệp vụ của dự án trước khi lựa chọn công nghệ.
- Thiết kế hệ thống AI Agent theo dạng mô-đun và tích hợp cơ chế xác nhận thông tin từ người dùng.
- Áp dụng nguyên lý MVP để nhanh chóng xây dựng các tính năng cốt lõi và kiểm thử tính hiệu quả của giải pháp.

### Hình ảnh sự kiện

![Thuyết trình FCAJ x Agentic AI Build Week](/images/4-EventParticipated/4.3-Event3/5b7d57fe-df8c-4710-8890-2d1d6832b154.jpg)

