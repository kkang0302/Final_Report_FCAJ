---
title : "Lưu trữ, CDN & Upload video"
date : 2024-01-01 
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

# Tích hợp Amazon S3, CloudFront CDN & Luồng Upload Video bảo mật

Hệ thống CourShare cần lưu trữ hai loại nội dung chính: mã nguồn tĩnh của ứng dụng (Frontend SPA) và các video bài giảng dung lượng lớn. Chúng ta sử dụng **Amazon S3** để lưu trữ và **Amazon CloudFront** làm mạng phân phối nội dung (CDN) giúp tối ưu hóa tốc độ tải trang và truyền tải video, kết hợp với cơ chế **S3 Presigned URL** để bảo mật luồng tải video lên từ Client.

## Bước 1: Khởi tạo và Cấu hình các S3 Buckets
Chúng ta tạo 2 buckets (chế độ Block all public access):
1. **Frontend Web Bucket (`courshare-frontend-web`):**
   - Dùng để lưu trữ toàn bộ code React build tĩnh.
2. **Media Bucket (`courshare-media-bucket`):**
   - Dùng để lưu trữ video gốc được upload lên và video bài học sau khi đã transcode định dạng HLS.
   - **Cấu hình CORS (Cross-Origin Resource Sharing):** Bắt buộc phải cấu hình CORS trên Media bucket để cho phép trình duyệt của Client gửi request HTTP PUT trực tiếp lên S3:
     ```json
     [
       {
         "AllowedHeaders": ["*"],
         "AllowedMethods": ["PUT", "GET", "HEAD"],
         "AllowedOrigins": ["*"],
         "ExposeHeaders": []
       }
     ]
     ```

## Bước 2: Phân phối qua Amazon CloudFront (OAC)
Để bảo mật tốt nhất cho website tĩnh, chúng ta tạo một CloudFront Distribution và cấu hình **Origin Access Control (OAC)** để ngăn chặn người dùng truy cập trực tiếp vào link S3:

1. Vào **CloudFront Console** -> **Create distribution**.
2. Origin domain: Chọn `courshare-frontend-web.s3.amazonaws.com`.
3. Origin access: Chọn **Origin access control settings (recommended)** -> Nhấp **Create control setting** -> Chọn **Sign requests (recommended)**.
4. Default cache behavior: Chọn **Redirect HTTP to HTTPS**.
5. Nhấp **Create distribution**.
6. **Cập nhật S3 Bucket Policy:** Sao chép mã Policy do CloudFront cấp và dán vào phần Bucket Policy của `courshare-frontend-web` để cho phép CloudFront OAC được phép đọc tệp tin:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": {
       "Sid": "AllowCloudFrontServicePrincipalReadOnly",
       "Effect": "Allow",
       "Principal": {
         "Service": "cloudfront.amazonaws.com"
       },
       "Action": "s3:GetObject",
       "Resource": "arn:aws:s3:::courshare-frontend-web/*",
       "Condition": {
         "StringEquals": {
           "AWS:SourceArn": "arn:aws:cloudfront::<AWS_ACCOUNT_ID>:distribution/<DISTRIBUTION_ID>"
         }
       }
     }
   }
   ```

## Bước 3: Luồng Upload Video bảo mật qua S3 Presigned URL
Nếu cho phép Client upload video trực tiếp thông qua máy chủ Backend (ví dụ Course Service), luồng dữ liệu truyền tải file nặng hàng trăm MB sẽ gây nghẽn băng thông và chiếm dụng tài nguyên CPU/RAM của container.

Giải pháp tối ưu là sử dụng **S3 Presigned URL**:

```
+--------+     1. Request Presigned URL      +----------------+
| Client | --------------------------------> | Course Service |
|        | <-------------------------------- |                |
+--------+       2. Return Presigned URL     +----------------+
    |
    | 3. HTTP PUT Upload Video trực tiếp
    v
+-----------+
| S3 Bucket |
+-----------+
```

### Triển khai mã nguồn sinh Presigned URL (Course Service - Spring Boot):
1. Khởi tạo Amazon S3 Client sử dụng AWS SDK:
   ```java
   AmazonS3 s3Client = AmazonS3ClientBuilder.standard()
           .withRegion(Regions.US_EAST_1)
           .build();
   ```
2. Sinh URL ký nhận:
   ```java
   java.util.Date expiration = new java.util.Date();
   long expTimeMillis = expiration.getTime() + 1000 * 60 * 15; // URL hết hạn trong 15 phút
   expiration.setTime(expTimeMillis);

   GeneratePresignedUrlRequest generatePresignedUrlRequest =
           new GeneratePresignedUrlRequest(bucketName, objectKey)
           .withMethod(HttpMethod.PUT)
           .withExpiration(expiration);
   
   URL url = s3Client.generatePresignedUrl(generatePresignedUrlRequest);
   return url.toString();
   ```
3. Backend trả URL này về cho Client. Client thực hiện request HTTP PUT chứa dữ liệu video gốc lên URL nhận được để hoàn tất upload trực tiếp lên S3 Media bucket mà không cần đi qua Backend.

<!-- TODO: chèn screenshot - [Màn hình AWS CloudFront Distributions hiển thị danh sách các phân phối đang hoạt động] -->

<!-- TODO: chèn screenshot - [Màn hình S3 Permissions tab hiển thị Bucket Policy được cấu hình chính xác cho CloudFront OAC Service Principal] -->

<!-- TODO: chèn screenshot - [Màn hình Console Chrome Network tab ghi nhận request PUT tải video trực tiếp lên S3 trả về mã trạng thái 200 OK] -->
