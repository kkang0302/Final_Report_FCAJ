---
title : "Dọn dẹp tài nguyên"
date : 2024-01-01 
weight : 12
chapter : false
pre : " <b> 5.12. </b> "
---

# Hướng dẫn Dọn dẹp Tài nguyên để Tránh phát sinh chi phí

Sau khi hoàn tất quá trình kiểm thử và demo dự án CourShare, việc dọn dẹp các tài nguyên đã khởi chạy trên AWS là vô cùng quan trọng nhằm tránh phát sinh các khoản chi phí ngoài ý muốn trên tài khoản của bạn. Hãy thực hiện xóa tài nguyên theo thứ tự dưới đây để đảm bảo không bị sót.

## Thứ tự dọn dẹp tài nguyên

### 1. Xóa các Dịch vụ ECS (ECS Services & Tasks)
* Truy cập **ECS Console** -> Chọn cluster `courshare-cluster` -> Chọn các service.
* Nhấp **Update Service** -> Đặt số lượng Tasks mong muốn về `0`.
* Sau khi các tasks ngừng chạy hoàn toàn, chọn các Service và nhấn **Delete**.
* Cuối cùng, xóa **ECS Cluster** `courshare-cluster`.

### 2. Xóa Application Load Balancer
* Vào **EC2 Console** -> **Load Balancers**.
* Chọn `courshare-alb` -> Nhấp **Actions** -> **Delete**.
* Tiếp theo, vào mục **Target Groups**, chọn các Target Groups tương ứng và nhấn **Delete**.

### 3. Xóa Cơ sở dữ liệu RDS
* Vào **RDS Console** -> **Databases**.
* Chọn instance `courshare-rds` -> **Actions** -> **Delete**.
* **Lưu ý:** Bỏ chọn tùy chọn "Create final snapshot" và chọn xác nhận đồng ý xóa vĩnh viễn.

### 4. Xóa NAT Instance (EC2)
* Vào **EC2 Console** -> **Instances**.
* Chọn `courshare-nat-instance` -> **Instance state** -> **Terminate instance**.

### 5. Xóa các S3 Buckets & CloudFront Distribution
* Vào **CloudFront Console**, chọn distribution đã tạo và nhấp **Disable**. Sau khi trạng thái chuyển sang disabled, nhấp **Delete**.
* Vào **S3 Console**, chọn các bucket `courshare-frontend-web` và `courshare-media-bucket` -> Nhấp **Empty** để xóa sạch các tệp tin lưu trữ bên trong trước, sau đó nhấp **Delete** để xóa bucket.

### 6. Xóa các tài nguyên khác
* **Cognito User Pool:** Vào Cognito Console -> Chọn `courshare-user-pool` -> Nhấp **Delete**.
* **Amazon SQS:** Vào SQS Console -> Chọn các queues -> Nhấp **Delete**.
* **Amazon ECR:** Vào ECR Console -> Chọn các repositories -> Nhấp **Delete**.
* **Cloud Map:** Vào Cloud Map Console -> Chọn Namespace `courshare.local` -> Nhấp **Delete**.
* **VPC:** Cuối cùng, vào VPC Console -> Chọn `courshare-vpc` -> Nhấp **Delete VPC**. (Việc này sẽ tự động xóa tất cả Subnets, Route Tables, Internet Gateway liên kết).

<!-- TODO: chèn screenshot - [Màn hình AWS VPC Console xác nhận đang xóa VPC và các thành phần liên quan thành công] -->
