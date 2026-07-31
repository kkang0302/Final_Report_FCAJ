---
title: "Blog 1"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Sự khác biệt giữa project thực tập đến production-ready: câu chuyện kiến trúc của CourShare

Khi bắt đầu xây CourShare - nền tảng vừa học vừa dạy, dùng chung một ví tiền cho cả hai vai trò - mục tiêu ban đầu của mình rất đơn giản: hiểu AWS vận hành như thế nào, và áp dụng nó vào một sản phẩm có ý nghĩa thật, không chỉ là bài lab rời rạc. Sau khi dựng xong kiến trúc microservices chạy trên ECS Fargate, mình có tự hỏi một câu: "Nếu kiến trúc này phải chịu tải thật, có người dùng thật, tiền thật chảy qua Stripe - nó đã đủ chuẩn chưa?"

Câu trả lời ngắn: chưa hẳn - và đó là một phát hiện thú vị hơn mình tưởng.

### Kiến trúc hiện tại: đủ tốt cho việc học, chưa đủ cho production

CourShare hiện tại gồm 5 microservice (Identity, Course, Enrollment, Learning, Payment) chạy trên ECS Fargate, đứng sau Application Load Balancer, giao tiếp nội bộ qua Cloud Map, mỗi service có database riêng theo schema, xác thực qua Cognito, video được upload thẳng lên S3 qua presigned URL rồi transcode sang HLS bằng một worker lắng nghe SQS, và toàn bộ pipeline deploy tự động qua GitHub Actions -> ECR -> ECS.

Nhìn tổng thể, đây là một kiến trúc đúng hướng: tách domain rõ ràng, xử lý bất đồng bộ cho tác vụ nặng (transcode video), tận dụng CDN cho nội dung tĩnh và streaming. Nhưng khi soi qua lăng kính AWS Well-Architected Framework, mình nhận ra một điều: phần lớn quyết định thiết kế đang nghiêng hẳn về tối ưu chi phí, mà đánh đổi lại là độ tin cậy và bảo mật - điều hoàn toàn hợp lý cho một project sinh viên/thực tập, nhưng sẽ là rủi ro nếu bê nguyên lên production.

### Ba điểm mình sẽ đổi đầu tiên nếu đưa CourShare lên production

#### 1. Bỏ NAT Instance, chuyển sang NAT Gateway
Hiện tại, toàn bộ private subnet (nơi các microservice chạy) đi ra internet qua một NAT Instance t4g.nano - một con EC2 nhỏ, rẻ, nhưng là một điểm chết duy nhất (single point of failure). Nếu instance đó gặp sự cố, mọi service trong private subnet mất kết nối internet: ECS không pull được image mới, Payment Service không gọi được Stripe API, video worker không tải/ghi được lên S3.

NAT Gateway của AWS giải quyết đúng vấn đề này - nó là dịch vụ managed, tự động high-availability trong một AZ, không cần mình tự vá lỗi hay theo dõi uptime. Cái giá phải trả là chi phí cao hơn NAT Instance khá nhiều, nhưng đổi lại là ngủ ngon hơn khi có traffic thật.

#### 2. Thêm lớp bảo vệ ở tầng biên: WAF, Secrets Manager, mã hoá dữ liệu
Hiện tại API Gateway của CourShare expose thẳng ra internet mà chưa có AWS WAF đứng chặn phía trước - nghĩa là chưa có lớp lọc request độc hại, chưa có rate limiting chống brute-force hay bot scraping. Ngoài ra, các connection string database và Stripe secret key cần được chuyển hẳn vào AWS Secrets Manager thay vì biến môi trường thuần, và cần xác nhận encryption at rest đã được bật cho cả RDS và S3 Media Bucket - vì video bài giảng và dữ liệu người dùng đều là tài sản cần bảo vệ.

### Những cải tiến tiếp theo, khi hệ thống đã có traffic thật

Ngoài ra mình đã suy nghĩ thêm vài hướng cải thiện dài hơi hơn:
* **Auto Scaling cho ECS Service** - để hệ thống tự thêm task khi có đợt học viên upload/xem video đông, và scale về mức thấp khi ít traffic (vừa chịu tải tốt hơn, vừa không tốn tiền chạy dư công suất lúc vắng khách).
* **Cache layer (ví dụ Redis/ElastiCache)** cho những dữ liệu đọc nhiều, ít đổi - như danh sách khoá học nổi bật - để giảm áp lực trực tiếp lên RDS.
* **Infrastructure as Code (Terraform/CDK)** - toàn bộ hạ tầng hiện đang dựng thủ công qua console; khi hệ thống lớn hơn, đây sẽ là cách duy nhất để thay đổi hạ tầng an toàn, có review, có rollback, thay vì "nhớ mình đã bấm gì trong console".
* **CloudWatch Alarms + Dashboard** thay vì chỉ gửi log để xem lại - mục tiêu là biết sự cố ngay lúc nó xảy ra, không phải sau khi người dùng report.
* **S3 Lifecycle Policy cho Media Bucket** - video gốc dung lượng lớn, nếu không có chính sách chuyển sang storage rẻ hơn (Glacier) hoặc dọn định kỳ, chi phí lưu trữ sẽ âm thầm phình to theo thời gian.

### Điều mình rút ra

Cái hay nhất của việc soi lại kiến trúc CourShare qua Well-Architected Framework không phải là danh sách "phải sửa gì", mà là nhận ra: mọi quyết định kiến trúc đều là một sự đánh đổi có chủ đích, không có kiến trúc nào "đúng tuyệt đối" - chỉ có kiến trúc phù hợp với đúng giai đoạn và đúng ràng buộc (ngân sách, mục tiêu học tập, quy mô người dùng thật). Việc dùng NAT Instance thay NAT Gateway, hay Single-AZ thay Multi-AZ, không phải là làm sai - mà là lựa chọn hợp lý cho một bản demo, miễn là mình biết rõ mình đang đánh đổi cái gì, và có sẵn một lộ trình để nâng cấp khi cần.

Đó cũng là lý do mình nghĩ bài tập này đáng để chia sẻ: không phải để nói "kiến trúc của tôi hoàn hảo", mà để cho thấy quá trình tự đặt câu hỏi "nếu phải chịu tải thật, phần nào sẽ gãy trước?" - và đó, với mình, mới là phần thú vị nhất khi học AWS.

---
*Xem bài viết chi tiết tại:* [AWS Study Group - Nhóm FB](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2229312141167079/?rdid=LvHCFBM57fzgRzFP#)