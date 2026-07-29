---
title : "Thanh toán & Ví tiền"
date : 2024-01-01 
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

# Triển khai Payment Service & Cơ chế Ví điện tử nội bộ tích hợp Stripe

Trái tim kinh tế của CourShare là **Payment Service** (Node.js/Express, kết nối tới RDS PostgreSQL schema `payment_db`). Dịch vụ này xử lý nạp/rút tiền thật bằng cổng thanh toán **Stripe** và quản lý các giao dịch ví-sang-ví nội bộ khi mua bán khóa học.

## 1. Thiết kế cơ sở dữ liệu Ví chung
Chúng ta thiết kế 2 bảng dữ liệu quan trọng trong schema `payment_db` (sử dụng Prisma ORM):
* Bảng `wallets`: Lưu trữ số dư ví (`balance`) của từng người dùng (`userId`). Một tài khoản duy nhất chứa cả số dư nạp vào và số dư thu nhập kiếm được từ việc bán khóa học.
* Bảng `transactions`: Ghi nhận lịch sử giao dịch. Các loại giao dịch gồm: `DEPOSIT` (nạp tiền qua Stripe), `WITHDRAW` (rút tiền về ngân hàng), `PURCHASE` (mua khóa học - trừ tiền người mua), và `EARN` (bán khóa học - cộng tiền cho giảng viên).

## 2. Luồng Nạp tiền (Stripe Integration)
Khi người dùng muốn nạp tiền thật vào ví nội bộ:

1. **Tạo Checkout Session:** Client gọi API tới Payment Service, backend sử dụng Stripe SDK để khởi tạo một trang thanh toán:
   ```javascript
   const session = await stripe.checkout.sessions.create({
     payment_method_types: ['card'],
     line_items: [{
       price_data: {
         currency: 'usd',
         product_data: { name: 'Nạp tiền ví CourShare' },
         unit_amount: amount * 100, // Đơn vị cent
       },
       quantity: 1,
     }],
     mode: 'payment',
     success_url: `${process.env.FRONTEND_URL}/payment/success`,
     cancel_url: `${process.env.FRONTEND_URL}/payment/cancel`,
     metadata: { userId }, // Truyền userId để nhận diện khi webhook gọi về
   });
   ```
2. Backend trả về `session.url` và client chuyển hướng người dùng sang trang thanh toán bảo mật của Stripe.

## 3. Stripe Webhook Endpoint (Cập nhật số dư)
Sau khi người dùng điền thẻ ngân hàng và thanh toán thành công, Stripe tự động gửi một HTTP POST request (Webhook) về API Gateway/Payment Service tại endpoint `/webhook`.

Endpoint này xử lý:
1. **Xác thực chữ ký:** Bảo mật webhook bằng cách kiểm tra chữ ký gửi từ Stripe nhằm tránh giả mạo:
   ```javascript
   const sig = req.headers['stripe-signature'];
   const event = stripe.webhooks.constructEvent(req.body, sig, process.env.WEBHOOK_SECRET);
   ```
2. **Xử lý sự kiện:** Khi nhận event `checkout.session.completed`:
   - Trích xuất thông tin `userId` và số tiền `amount` từ metadata.
   - Chạy một SQL transaction trong PostgreSQL: Cộng tiền vào bảng `wallets` của user, đồng thời tạo một bản ghi `DEPOSIT` mới trong bảng `transactions`.

## 4. Luồng Mua khóa học (Ví-sang-Ví nội bộ)
Khi học viên quyết định mua khóa học từ giảng viên:
1. Client gửi request chứa `courseId` và `instructorId` lên Payment Service.
2. Payment Service thực hiện SQL transaction:
   - Kiểm tra số dư ví người mua. Nếu nhỏ hơn giá khóa học, từ chối giao dịch.
   - Trừ tiền ví người mua.
   - Cộng tiền ví người dạy (`instructorId`).
   - Ghi nhận 2 bản ghi lịch sử tương ứng: `PURCHASE` (người mua) và `EARN` (người dạy).
3. **Đảm bảo tính nhất quán (Eventual Consistency):**
   Sau khi transaction database hoàn tất, Payment Service bắn một message chứa `userId` và `courseId` vào hàng đợi SQS. **Enrollment Service** lắng nghe queue này, tự động đăng ký và mở khóa quyền truy cập bài học cho học viên. Luồng này giúp hệ thống hoạt động ổn định và tin cậy ngay cả khi các dịch vụ khác gặp sự cố tạm thời.

<!-- TODO: chèn screenshot - [Màn hình trang thanh toán của Stripe hiển thị thông tin hóa đơn nạp tiền ví CourShare] -->

<!-- TODO: chèn screenshot - [Màn hình Terminal chạy Stripe CLI forwarding webhook events tới local port 8084 để phát triển và debug] -->

<!-- TODO: chèn screenshot - [Màn hình cơ sở dữ liệu hiển thị bảng wallets ghi nhận số dư ví của học viên bị trừ và ví giảng viên được cộng chính xác sau giao dịch] -->
