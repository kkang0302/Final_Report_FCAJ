---
title : "Giới thiệu"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

# Tổng quan kiến trúc CourShare trên AWS

Trong chuỗi bài thực hành này, chúng ta sẽ xây dựng và triển khai **CourShare** — một nền tảng chia sẻ và học tập trực tuyến tích hợp ví điện tử dùng chung. Dự án được triển khai trên đám mây AWS sử dụng kiến trúc Microservices container chạy trên cụm serverless container **AWS ECS Fargate**, đảm bảo hiệu năng cao, bảo mật chặt chẽ và khả năng co giãn linh hoạt.

## Sơ đồ kiến trúc tổng thể

Dưới đây là sơ đồ kiến trúc hạ tầng AWS hoàn chỉnh của hệ thống CourShare:

![Sơ đồ kiến trúc CourShare](/images/AWS_CourShare.drawio.png)

## Các luồng vận hành chính

1. **Truy cập Client & CDN:** Người dùng (Học viên/Giảng viên) truy cập vào ứng dụng Single Page Application (SPA React) được lưu trữ trên **Amazon S3 Frontend Web Bucket**, phân phối nhanh chóng qua mạng lưới CDN **Amazon CloudFront**.
2. **Xác thực người dùng:** Người dùng thực hiện đăng ký/đăng nhập trực tiếp với **Amazon Cognito User Pool**. Sau khi thành công, client nhận về JWT Access Token để đính kèm vào header của các API requests.
3. **Định tuyến API:** Các API request đi qua CloudFront CDN sẽ được chuyển tiếp tới **Amazon API Gateway** (đóng vai trò ngược lại proxy hoặc gateway định tuyến phía trước). API Gateway chuyển tiếp các request tới **Application Load Balancer (ALB)**.
4. **Xử lý tại Microservices:** ALB thực hiện path-based routing (định tuyến theo đường dẫn) để chuyển request vào cụm **AWS ECS Fargate Cluster** nằm trong Private Subnets an toàn:
   - Request `/auth/**` chuyển tới **Identity Service** (port 8080) để quản lý hồ sơ người dùng.
   - Request `/courses/**` chuyển tới **Course Service** (port 8081) để quản lý nội dung giảng dạy.
   - Request `/enrollments/**` chuyển tới **Enrollment Service** (port 8082) để xử lý đăng ký lớp học.
   - Request `/learning/**` chuyển tới **Learning Service** (port 8083) để theo dõi tiến độ bài giảng.
   - Request `/payment/**` chuyển tới **Payment Service** (port 8084) để quản lý số dư ví và tích hợp Stripe thanh toán nạp/rút tiền thật.
5. **Giao tiếp nội bộ (Service Discovery):** Các service giao tiếp với nhau qua DNS name nội bộ (ví dụ: `http://course-service.courshare.local:8081`) nhờ **Amazon Cloud Map**, tránh làm lộ API ra Internet công cộng.
6. **Lưu trữ dữ liệu:** Mỗi microservice có cơ sở dữ liệu biệt lập theo từng schema độc lập nằm trên một máy chủ cơ sở dữ liệu quan hệ **Amazon RDS PostgreSQL** chung nhằm tối ưu chi phí nhưng vẫn đảm bảo tính độc lập dữ liệu.
7. **Xử lý media và tác vụ bất đồng bộ:**
   - Giảng viên yêu cầu upload video sẽ nhận về **S3 Presigned URL** và tải trực tiếp file lên **Amazon S3 Media Bucket**.
   - Sự kiện upload video mới kích hoạt đẩy thông tin vào hàng đợi **Amazon SQS Queue**.
   - **VideoWorker** lắng nghe queue, tải video gốc từ S3, chuyển đổi định dạng sang HLS (transcode m3u8/ts segments) và lưu ngược lại S3 để streaming.
   - **NotificationWorker** lắng nghe sự kiện từ SQS và thực hiện gửi email thông báo giao dịch thành công cho người dùng qua dịch vụ gửi email.
8. **Giám sát & CI/CD:** Toàn bộ log của các container microservices được đẩy về **Amazon CloudWatch Logs**. Mỗi khi có mã nguồn mới trên **GitHub**, quy trình CI/CD **GitHub Actions** tự động build Docker image, đẩy lên **Amazon ECR** và cập nhật ECS Service tự động.
