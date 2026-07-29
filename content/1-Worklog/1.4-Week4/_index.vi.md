---
title: "Worklog Tuần 4"
date: 2026-07-06
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

* Thiết kế và tự động hóa toàn bộ hạ tầng mạng, bảo mật và cơ sở dữ liệu trên AWS thông qua Terraform (Infrastructure as Code).
* Xây dựng NAT Instance để tối ưu hóa chi phí thay cho AWS NAT Gateway thông thường.
* Thiết lập hệ thống cơ sở dữ liệu chia sẻ Amazon RDS PostgreSQL trong phân vùng mạng cô lập.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Viết tệp cấu hình Terraform (`main.tf`) định nghĩa Virtual Private Cloud (VPC) với dải IP `10.0.0.0/16` | 06/07/2026 | 06/07/2026 | Terraform AWS Provider Docs |
| 3 | - Thiết lập cấu trúc mạng gồm Public, Private và Database Subnets phân bổ trên 2 Availability Zones (`ap-southeast-1a`, `ap-southeast-1b`) | 07/07/2026 | 07/07/2026 | AWS VPC Subnet Design Guidelines |
| 4 | - Khởi tạo EC2 NAT Instance (`t3.micro`, Amazon Linux 2023) trong Public Subnet và viết script `user_data` kích hoạt chuyển tiếp gói tin, định tuyến cho Private Subnet | 08/07/2026 | 08/07/2026 | NAT Instance configuration guides |
| 5 | - Khởi tạo Amazon RDS PostgreSQL (`db.t4g.micro`) trong Database Subnets cô lập an toàn | 09/07/2026 | 09/07/2026 | Terraform RDS Resource Guide |
| 6 | - Cấu hình Security Group cổng `5432` chỉ cho phép truy cập nội bộ từ ECS và NAT Instance; chạy thử `terraform apply` kiểm chứng | 10/07/2026 | 10/07/2026 | AWS Security Group Rules |

### Kết quả đạt được tuần 4:

* Thiết lập thành công hạ tầng AWS an toàn tại region Singapore hoàn toàn bằng Terraform.
* Thiết lập NAT Instance hoạt động tốt, giúp các máy ảo/container mạng Private truy cập Internet để cài thư viện mà không bị phơi lộ cổng ra ngoài Internet, tiết kiệm đáng kể chi phí hạ tầng.
* Cơ sở dữ liệu RDS PostgreSQL chạy ổn định, cách ly an toàn trong database subnet riêng biệt.

![VPC Map](/images/1-WorkLog/VPC.png)
*Sơ đồ Resource Map của VPC CourShare hiển thị trực quan các phân vùng mạng.*

![VPC Subnets](/images/1-WorkLog/Subnet.png)
*Danh sách các subnet (Public, Private, Database) được tạo lập trên AWS Console.*

![VPC Route Tables](/images/1-WorkLog/Route-table.png)
*Bảng định tuyến của các subnet liên kết qua Internet Gateway hoặc NAT Instance.*

![EC2 NAT Instance](/images/1-WorkLog/EC2.png)
*NAT Instance chạy trên máy ảo EC2 t3.micro ở trạng thái Running.*

![RDS Database](/images/1-WorkLog/RDS.png)
*Amazon RDS PostgreSQL instance ở trạng thái Available.*

