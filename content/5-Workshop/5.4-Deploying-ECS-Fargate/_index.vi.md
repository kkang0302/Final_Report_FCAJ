---
title : "Triển khai Fargate & ALB"
date : 2024-01-01 
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

# Triển khai Microservices với AWS ECS Fargate & ALB

Chúng ta sẽ đóng gói 5 microservice và 2 workers của CourShare thành các Docker image, đẩy lên Amazon ECR và chạy dưới dạng tasks trong cụm serverless container AWS ECS Fargate, sau đó cấu hình định tuyến thông qua Application Load Balancer.

## Bước 1: Tạo Amazon ECR Repositories
1. Truy cập **Amazon ECR** Console -> **Repositories** -> **Create repository**.
2. Tạo 7 repositories tương ứng với các dịch vụ (chọn chế độ Private):
   - `courshare-identity-service`
   - `courshare-course-service`
   - `courshare-enrollment-service`
   - `courshare-learning-service`
   - `courshare-payment-service`
   - `courshare-video-worker`
   - `courshare-notification-worker`

## Bước 2: Build & Push Images lên ECR (Thử nghiệm thủ công)
1. Đăng nhập Docker client vào ECR Registry của bạn:
   ```bash
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
   ```
2. Di chuyển vào thư mục code của từng service (ví dụ: Identity Service), build Docker image:
   ```bash
   docker build -t courshare-identity-service .
   docker tag courshare-identity-service:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/courshare-identity-service:latest
   docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/courshare-identity-service:latest
   ```
   *(Thực hiện tương tự cho các microservice và worker còn lại).*

## Bước 3: Tạo Application Load Balancer (ALB)
1. Vào **EC2 Console** -> **Load Balancers** -> **Create Load Balancer** -> Chọn **Application Load Balancer**.
2. Tên ALB: `courshare-alb`.
3. Network Mapping:
   - VPC: `courshare-vpc`
   - Subnets: Chọn `courshare-public-subnet-1` và `2`.
4. Security Groups: Chọn `courshare-alb-sg`.
5. Listeners and Routing:
   - Tạo 1 Listener port `80` (HTTP) trỏ về Target Group mặc định.
6. Click **Create load balancer**.

## Bước 4: Soạn thảo Task Definitions
Với mỗi microservice, tạo một Task Definition mới trên cụm ECS:
1. Chọn launch type: **AWS Fargate**.
2. Task size: CPU: `0.25 vCPU`, Memory: `0.5 GB` (Tối thiểu để tiết kiệm chi phí).
3. Task Role & Task Execution Role: Chọn `ecsTaskExecutionRole` (đã được gán policy cho phép pull image từ ECR và ghi log về CloudWatch).
4. Container Definitions:
   - Name: ví dụ `identity-service`.
   - Image URI: `<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/courshare-identity-service:latest`.
   - Port Mapping: container port `8080` (Identity) / `8081` (Course) / `8082` (Enrollment) / `8083` (Learning) / `8084` (Payment).

## Bước 5: Tạo các Target Groups & Cấu hình Routing
1. Tạo 5 Target Groups riêng tương ứng với 5 dịch vụ (Target type: **IP**, Protocol: **HTTP**, Port: Cổng tương ứng của service).
2. Vào **ALB Listener Rules** của Port 80, cấu hình định tuyến theo Path (Path-based routing):
   - Đường dẫn `/auth/**` -> Forward tới `tg-identity-service`.
   - Đường dẫn `/courses/**` -> Forward tới `tg-course-service`.
   - Đường dẫn `/enrollments/**` -> Forward tới `tg-enrollment-service`.
   - Đường dẫn `/learning/**` -> Forward tới `tg-learning-service`.
   - Đường dẫn `/payment/**` -> Forward tới `tg-payment-service`.

## Bước 6: Khởi tạo ECS Services & Cấu hình Service Discovery
1. Tạo **ECS Cluster** tên là `courshare-cluster`.
2. Tạo các ECS Services cho 5 microservices:
   - Launch type: **Fargate**.
   - Service Name: ví dụ `identity-service`.
   - Number of tasks: `1`.
   - Network Settings:
     - Subnets: Chọn `courshare-private-subnet-1` và `2`.
     - Security Group: Chọn `courshare-ecs-tasks-sg`.
     - Auto-assign public IP: **Disabled**.
   - Load Balancing: Chọn `courshare-alb`, liên kết với Target Group tương ứng.
3. **Cấu hình Service Discovery (Amazon Cloud Map):**
   - Bật tùy chọn **Service Discovery**.
   - Namespace: Chọn `courshare.local`.
   - Service discovery name: ví dụ `identity-service`.
   - Giờ đây, các service có thể gọi nhau nội bộ qua DNS như: `http://identity-service.courshare.local:8080`.
4. Tạo thêm 2 ECS Services cho `VideoWorker` và `NotificationWorker` (không cần cấu hình Load Balancer vì đây là các daemon worker chỉ lắng nghe SQS).

<!-- TODO: chèn screenshot - [Màn hình Amazon ECR Console hiển thị danh sách 7 repository private của dự án] -->

<!-- TODO: chèn screenshot - [Màn hình AWS ECS Console hiển thị danh sách các service trong cụm cluster đều có trạng thái Active và Running Tasks = 1] -->

<!-- TODO: chèn screenshot - [Màn hình AWS Route 53 Private Hosted Zone hiển thị các bản ghi DNS .local được tạo tự động bởi Cloud Map] -->
