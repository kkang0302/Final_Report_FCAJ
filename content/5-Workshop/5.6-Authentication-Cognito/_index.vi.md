---
title : "Xác thực với Cognito"
date : 2024-01-01 
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

# Quản lý Xác thực và Định danh người dùng với Amazon Cognito

Hệ thống CourShare quản lý định danh người dùng tập trung bằng **Amazon Cognito User Pool**. Chúng ta cấu hình một cơ chế xác thực duy nhất cho cả hai vai trò: học viên và giảng viên, giúp người dùng linh hoạt học tập và chia sẻ kiến thức trên cùng một tài khoản.

## Bước 1: Khởi tạo Cognito User Pool
1. Truy cập **Cognito Console** -> **Create user pool**.
2. **Configure sign-in experience:**
   - Chọn **Cognito user pool**.
   - Sign-in providers: Chọn **Email** (để người dùng đăng nhập bằng địa chỉ email).
3. **Configure security requirements:**
   - Password policy: Giữ mặc định hoặc tùy chỉnh độ phức tạp.
   - Multi-factor authentication (MFA): Chọn **No MFA** cho môi trường thử nghiệm (hoặc chọn Optional).
   - User recovery: Chọn **Email only**.
4. **Configure sign-up experience:**
   - Kích hoạt **Self-service sign-up** (cho phép người dùng tự đăng ký).
   - Attribute verification: Chọn **Send email message, verify email address**.
5. **Configure message delivery:**
   - Email: Chọn **Send email with Cognito** để sử dụng tài khoản email mặc định của AWS (giới hạn 50 email/ngày, tối ưu cho thử nghiệm).

## Bước 2: Thêm Thuộc tính Tùy chỉnh (custom:role)
Để hỗ trợ mô hình ví chung đa vai trò (Student và Instructor trên cùng 1 tài khoản), chúng ta tạo một thuộc tính tùy chỉnh lưu vai trò của người dùng:
1. Tại bước **Configure attributes**, nhấp vào **Add custom attribute**.
2. Tên thuộc tính: `role`.
3. Kiểu dữ liệu: **String**.
4. Min length: `1`, Max length: `20`.
5. Đảm bảo thuộc tính này có thể ghi (mutable) để giảng viên có thể cập nhật vai trò sau này.

## Bước 3: Cấu hình App Client
1. Tại bước **Integrate app**:
   - User pool name: `courshare-user-pool`.
   - App client name: `courshare-react-spa`.
   - Client secret: Chọn **Don't generate a client secret** (Bắt buộc phải chọn tùy chọn này vì Frontend React Single Page App chạy trên trình duyệt client, không thể bảo mật Client Secret an toàn).
2. Hoàn tất các bước và nhấp **Create user pool**.

## Bước 4: Tích hợp và Xác thực JWT Token tại Identity Service
Khi client React đăng nhập thành công với Cognito, Cognito sẽ trả về 3 tokens: ID Token, Access Token, và Refresh Token.
Client đính kèm Access Token (JWT) vào header: `Authorization: Bearer <JWT_TOKEN>` khi gọi API.

Identity Service (Java Spring Boot) thực hiện kiểm tra chữ ký token bằng cách cấu hình Spring Security:
1. Spring Boot lấy cấu hình JWK Set URI từ Cognito:
   ```yaml
   spring:
     security:
       oauth2:
         resourceserver:
           jwt:
             issuer-uri: https://cognito-idp.us-east-1.amazonaws.com/us-east-1_xxxxxx
             jwk-set-uri: https://cognito-idp.us-east-1.amazonaws.com/us-east-1_xxxxxx/.well-known/jwks.json
   ```
2. Thư viện Spring Security tự động tải các public keys từ Cognito, kiểm tra tính hợp lệ và thời hạn của token.
3. Chiết xuất các claims như `username`, `email` và `custom:role` từ token để thực hiện phân quyền.

<!-- TODO: chèn screenshot - [Màn hình Amazon Cognito User Pool Console hiển thị thông tin tổng quan User Pool ID và App Client ID] -->

<!-- TODO: chèn screenshot - [Màn hình cấu hình App Client Attributes cho phép app client React đọc và ghi thuộc tính custom:role] -->
