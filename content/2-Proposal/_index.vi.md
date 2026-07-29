---
title: "Bản đề xuất"
date: 2026-07-28
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# CourShare - Nền tảng Chia sẻ và Học tập Trực tuyến Tích hợp
## Giải pháp Kiến trúc Microservices Containerized chạy trên AWS ECS Fargate

### 1. Tóm tắt điều hành
**CourShare** là một nền tảng học tập trực tuyến kết hợp các ưu điểm của mô hình chia sẻ video (tương tự YouTube) và các khóa học có cấu trúc bài bản (tương tự Udemy/Coursera). Tuy nhiên, CourShare thu hẹp phạm vi nội dung để tập trung hoàn toàn vào video bài giảng phục vụ mục đích học tập và phát triển tri thức, tránh sự phân tâm bởi các nội dung giải trí đa chủ đề. 

Nền tảng được thiết kế chạy trên hạ tầng đám mây AWS nhằm tối ưu hóa khả năng vận hành, độ tin cậy và khả năng mở rộng. Bằng cách áp dụng kiến trúc Microservices chạy trên môi trường Container (AWS ECS Fargate), CourShare không chỉ đáp ứng tốt nhu cầu học tập trực tuyến hiệu năng cao mà còn tạo ra một môi trường thực hành toàn diện, vận dụng tối đa các dịch vụ hiện đại của AWS để giải quyết bài toán thực tế.

### 2. Tuyên bố vấn đề
* **Vấn đề hiện tại:**
  Trên các nền tảng học tập trực tuyến phổ biến hiện nay, vai trò của "người học" và "người dạy" thường bị tách biệt rạch ròi. Người dùng phải đăng ký các loại tài khoản khác nhau hoặc chuyển đổi qua lại giữa các giao diện quản lý rất phức tạp. Nghiêm trọng hơn, ví tiền và luồng thanh toán của hai vai trò này cũng riêng biệt, gây ra sự bất tiện lớn khi giảng viên muốn dùng số dư kiếm được từ việc bán khóa học của mình để mua và học các khóa học khác trên cùng một nền tảng (họ phải rút tiền về ngân hàng cá nhân, chịu phí giao dịch, rồi nạp lại tiền vào tài khoản để mua khóa học khác).

* **Giải pháp từ CourShare:**
  CourShare giới thiệu mô hình **Ví điện tử chung (shared wallet)** cho mỗi người dùng tích hợp ngay trong một tài khoản duy nhất. Bất kỳ ai đăng ký tài khoản đều có thể đóng vai trò kép: vừa đăng tải khóa học của mình để chia sẻ/bán, vừa có thể tìm kiếm, mua và học các khóa học từ người khác.
  - Khi có học viên mua khóa học của bạn, tiền bán được lập tức cộng vào ví điện tử nội bộ.
  - Khi bạn mua khóa học của người khác, tiền sẽ được trừ trực tiếp từ số dư ví đó.
  - Cơ chế này tạo ra một **nền kinh tế tri thức nội bộ (internal creator economy)**, khuyến khích học tập suốt đời (lifelong learning), giảm thiểu tối đa ma sát thanh toán và đơn giản hóa quy trình kế toán giao dịch ở quy mô cá nhân hoặc doanh nghiệp nhỏ.

* **Lợi ích và hoàn vốn đầu tư (ROI):**
  - **Về mặt kỹ thuật:** Giúp đội ngũ kỹ sư thực hành thiết kế hệ thống phân tán, xử lý tính nhất quán dữ liệu giữa microservice thanh toán (Payment Service) và microservice đăng ký học (Enrollment Service), quản lý xử lý media bất đồng bộ và tối ưu hóa hạ tầng AWS.
  - **Về mặt vận hành:** Giảm thiểu chi phí giao dịch qua các cổng thanh toán bên thứ ba đối với các giao dịch mua bán nội bộ nhờ cơ chế ví chung. Hệ thống tận dụng AWS Free Tier và NAT Instance giá rẻ để duy trì chi phí vận hành ở mức tối thiểu trong giai đoạn phát triển và chạy thử nghiệm.

### 3. Kiến trúc giải pháp
Kiến trúc hệ thống CourShare được xây dựng theo mô hình Microservices, chạy trên nền tảng Serverless Container (ECS Fargate) nhằm đảm bảo tính bảo mật độc lập giữa các dịch vụ, khả năng tự động co giãn và giảm thiểu gánh nặng quản lý máy chủ.

![CourShare Architecture](/images/AWS_CourShare.drawio.png)

#### Dịch vụ AWS sử dụng
* **Amazon CloudFront (CDN):** Phân phối nội dung tĩnh của website (SPA Frontend) và stream video HLS hiệu năng cao với độ trễ thấp trên toàn cầu.
* **Amazon S3:**
  - *Frontend Web Bucket:* Lưu trữ mã nguồn tĩnh của ứng dụng React.
  - *Media Bucket:* Lưu trữ video bài học gốc và các video đã transcode định dạng HLS.
* **Amazon API Gateway:** Cổng tiếp nhận duy nhất cho các API request từ Client, thực hiện điều hướng các request đến Application Load Balancer (ALB).
* **Amazon VPC:** Phân vùng mạng độc lập, an toàn bao gồm:
  - *Public Subnet:* Chứa Application Load Balancer và NAT Instance (t4g.nano) để cung cấp Internet đi ra cho Private Subnet với chi phí cực thấp.
  - *Private Subnet:* Chứa cụm máy chủ AWS ECS Cluster (Fargate) chạy các container microservice và worker.
  - *Database Subnet:* Phân vùng cô lập hoàn toàn cho các cơ sở dữ liệu Amazon RDS PostgreSQL.
* **AWS ECS Fargate:** Chạy các microservice (Identity, Course, Enrollment, Learning, Payment) và các Worker (VideoWorker, NotificationWorker) dưới dạng Docker containers.
* **Amazon Cloud Map:** Giải pháp Service Discovery hỗ trợ các microservice trong mạng VPC private giao tiếp nội bộ với nhau qua DNS name thay vì hard-code IP.
* **Amazon Cognito:** Quản lý định danh người dùng (Student/Instructor) tập trung, cung cấp cơ chế phân quyền an toàn.
* **Amazon RDS (PostgreSQL):** Hệ quản trị cơ sở dữ liệu quan hệ cho các dịch vụ, thiết lập tách biệt schema theo từng service (IdentityDB, CourseDB, EnrollmentDB, LearningDB, PaymentDB) nhằm đảm bảo tính độc lập dữ liệu.
* **Amazon SQS Queue:** Hàng đợi tin nhắn trung gian để gửi các tác vụ xử lý bất đồng bộ (transcode video, gửi email thông báo).
* **Amazon CloudWatch:** Thu thập log tập trung từ toàn bộ ECS Tasks và cơ sở dữ liệu để giám sát và cảnh báo hệ thống.
* **CI/CD Pipeline (GitHub Actions & Amazon ECR):** Tự động hóa quá trình đóng gói Docker image, push lên Amazon ECR và cập nhật triển khai lên ECS Fargate.

#### Thiết kế thành phần
* **Identity Service (Port 8080 - Java Spring Boot):** Quản lý hồ sơ người dùng và tương tác với Amazon Cognito để xác thực và cấp JWT Token.
* **Course Service (Port 8081 - Java Spring Boot):** Quản lý thông tin khóa học, danh mục, bài giảng, và tạo liên kết tải video qua Presigned URL.
* **Enrollment Service (Port 8082 - Node.js/Express):** Quản lý lịch sử đăng ký khóa học của học viên.
* **Learning Service (Port 8083 - Node.js/Express):** Theo dõi tiến độ học tập, trạng thái hoàn thành bài học của người dùng.
* **Payment Service (Port 8084 - Node.js/Express):** Tích hợp cổng thanh toán Stripe thực tế để xử lý nạp/rút tiền thật vào ví điện tử nội bộ, quản lý số dư ví và lịch sử giao dịch.
* **VideoWorker (Node.js):** Lắng nghe tin nhắn từ SQS, tải video gốc từ S3, thực hiện transcode sang định dạng HLS (m3u8/ts) và đẩy ngược lại S3 để streaming.
* **NotificationWorker (Node.js):** Lấy tin nhắn từ SQS để gửi email thông báo mua khóa học thành công, cập nhật tiến độ học tập cho người dùng.

### 4. Triển khai kỹ thuật
#### Các giai đoạn triển khai
1. **Nghiên cứu & Thiết kế Kiến trúc (Tháng 1):** Phân tích nghiệp vụ CourShare, xây dựng sơ đồ thực thể dữ liệu (ERD) cho từng microservice và phác thảo kiến trúc hạ tầng AWS.
2. **Ước tính ngân sách & Phân tích tính khả thi (Tháng 1):** Tính toán chi phí thông qua AWS Pricing Calculator, lựa chọn NAT Instance thay vì NAT Gateway để tiết kiệm tối đa ngân sách.
3. **Thiết lập và Tối ưu hóa Infrastructure (Tháng 2):** Khởi tạo VPC, cấu hình mạng Subnet, xây dựng RDS PostgreSQL instances và Amazon Cognito User Pool.
4. **Phát triển, Đóng gói Container & Kiểm thử (Tháng 2 - Tháng 3):** Triển khai mã nguồn các microservices, viết Dockerfile cho từng service, cấu hình định tuyến ALB, tích hợp Stripe Webhook và xây dựng các Worker xử lý hàng đợi SQS.
5. **Thiết lập CI/CD & Giám sát (Tháng 3):** Tạo GitHub Actions workflow để tự động hóa deploy lên ECS Fargate và thiết lập CloudWatch Logs Dashboard.

#### Yêu cầu kỹ thuật
* **Môi trường phát triển:** Docker Desktop, Node.js (v18+), Java JDK 17, Prisma CLI, AWS CLI.
* **Cấu hình Microservices:**
  - Các service Java Spring Boot sử dụng Spring Security tích hợp JWT và kết nối PostgreSQL qua Spring Data JPA.
  - Các service Node.js sử dụng Express, Prisma ORM để quản lý và truy vấn dữ liệu PostgreSQL.
  - Stripe SDK được tích hợp trong Payment Service để lắng nghe Stripe Webhooks, xác thực giao dịch nạp tiền an toàn.

### 5. Lộ trình & Mốc triển khai
* **Tuần 1 - 2:** Nghiên cứu AWS Cloud, khởi dựng VPC và cấu hình phân vùng mạng an toàn.
* **Tuần 3 - 4:** Đóng gói container các dịch vụ, thiết lập ECS Fargate Cluster và cấu hình ALB để định tuyến API.
* **Tuần 5 - 6:** Dựng RDS PostgreSQL cho 5 cơ sở dữ liệu độc lập; cấu hình Cognito User Pool để xác thực người dùng tập trung.
* **Tuần 7 - 8:** Dựng S3 Frontend và S3 Media. Tạo phân phối CloudFront CDN. Cấu hình luồng sinh Presigned URL để upload video bảo mật từ client.
* **Tuần 9 - 10:** Tích hợp Amazon SQS, xây dựng VideoWorker transcode HLS và NotificationWorker gửi email. Hoàn thiện Payment Service với ví điện tử và tích hợp Stripe.
* **Tuần 11 - 12:** Tự động hóa CI/CD với GitHub Actions và Amazon ECR. Cấu hình giám sát logs qua CloudWatch. Thực hiện kiểm thử tích hợp toàn diện hệ thống.

### 6. Ước tính ngân sách
Bảng dưới đây trình bày ước tính chi phí hàng tháng đối với hệ thống CourShare vận hành ở quy mô thử nghiệm/demo (sử dụng tối đa AWS Free Tier kết hợp với cấu hình tài nguyên nhỏ nhất):

| Dịch vụ AWS | Cấu hình thử nghiệm | Chi phí ước tính / Tháng | Ghi chú |
| --- | --- | --- | --- |
| **AWS ECS Fargate** | 5 Services + 2 Workers (0.25 vCPU, 0.5 GB RAM mỗi task) | ~$15.00 | Sử dụng Fargate Spot để giảm 70% chi phí |
| **Amazon RDS** | PostgreSQL db.t4g.micro (20 GB SSD) | ~$10.00 | Tích hợp multi-schema chung trên 1 Instance để tiết kiệm |
| **VPC NAT Instance** | 1 instance t4g.nano làm NAT thay cho NAT Gateway | ~$3.20 | Giảm chi phí đáng kể so với NAT Gateway ($32/tháng) |
| **Amazon S3 & CDN** | Lưu trữ video và CDN CloudFront | ~$2.00 | Phụ thuộc dung lượng video upload và băng thông |
| **Amazon Cognito** | Dưới 50,000 người dùng kích hoạt | $0.00 | Nằm trong AWS Free Tier |
| **Amazon SQS & API GW**| Tần suất request thử nghiệm thấp | ~$0.50 | Nằm trong AWS Free Tier |
| **CloudWatch Logs** | Lưu trữ log dưới 5 GB | $0.00 | Nằm trong AWS Free Tier |
| **Tổng cộng** | | **~$30.70 / Tháng** | *Đây là ước tính tham khảo, người dùng có thể tùy chỉnh.* |

### 7. Đánh giá rủi ro
* **Rủi ro 1: Chi phí lưu trữ và băng thông video tăng đột biến**
  - *Mô tả:* Khi số lượng khóa học và video bài giảng tăng lên, chi phí lưu trữ S3 và băng thông truyền tải CDN CloudFront có thể tăng rất nhanh.
  - *Giảm thiểu:* Cấu hình vòng đời đối tượng (S3 Lifecycle) để chuyển video cũ sang Glacier; áp dụng các thuật toán nén video tốt trong VideoWorker; thiết lập giới hạn dung lượng video tối đa cho mỗi bài giảng (ví dụ 100MB).
* **Rủi ro 2: Mất nhất quán dữ liệu giữa Payment Service và Enrollment Service**
  - *Mô tả:* Lỗi xảy ra khi trừ tiền ví người mua thành công nhưng hệ thống gặp sự cố khiến học viên không được ghi danh vào khóa học.
  - *Giảm thiểu:* Áp dụng mô hình giao dịch bất đồng bộ thông qua Outbox Pattern hoặc hàng đợi SQS để đảm bảo xử lý nhất quán (eventual consistency) kèm cơ chế retry tự động.
* **Rủi ro 3: Tranh chấp bản quyền và nội dung không phù hợp**
  - *Mô tả:* Người dùng upload các video vi phạm bản quyền hoặc nội dung độc hại.
  - *Giảm thiểu:* Sử dụng Amazon Rekognition để lọc tự động nội dung hình ảnh/video phản cảm khi upload; cung cấp chức năng báo cáo vi phạm (report) để quản trị viên xử lý thủ công.
* **Rủi ro 4: Độ trễ xử lý chuyển đổi định dạng video (Transcoding)**
  - *Mô tả:* Việc transcode video sang HLS tốn nhiều thời gian và tài nguyên CPU, gây nghẽn hàng đợi SQS.
  - *Giảm thiểu:* Sử dụng ECS Auto Scaling để tăng số lượng VideoWorker tasks khi hàng đợi SQS có nhiều tin nhắn chờ xử lý; tối ưu tham số transcode của công cụ ffmpeg.

### 8. Kết quả kỳ vọng
* **Hạ tầng hoàn chỉnh:** Triển khai thành công hệ thống microservices tự động hóa hoàn toàn trên AWS ECS Fargate, bảo mật chặt chẽ trong VPC private.
* **Tính năng hoàn thiện:** Người dùng có thể đăng ký tài khoản, nạp tiền vào ví bằng Stripe, tự do đăng tải video bài giảng cá nhân, mua khóa học bằng ví chung và xem video stream HLS mượt mà.
* **Khả năng quan sát tốt:** Log và lỗi từ tất cả dịch vụ được theo dõi trực quan và cảnh báo kịp thời qua Amazon CloudWatch.
* **Pipeline mượt mà:** Mọi thay đổi mã nguồn trên GitHub được build tự động và cập nhật lên AWS chỉ trong vài phút thông qua CI/CD pipeline an toàn.