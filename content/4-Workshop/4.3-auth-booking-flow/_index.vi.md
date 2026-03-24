---
title: "Luồng xác thực và đặt chỗ"
date: 2025-10-13
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

## Tổng quan luồng

Giai đoạn xác thực và booking trải dài trên cả frontend lẫn backend. Ở phía client, người dùng đăng nhập, duyệt station, tải slot data, chọn vehicle và gửi yêu cầu booking. Ở phía server, hệ thống xác thực request, kiểm tra quyền giữ slot, tạo booking, xác nhận booking ở bước sau và trả về QR image cho giai đoạn vận hành tiếp theo.

## Các điểm implementation quan trọng

- registration dùng OTP verification trước khi đi vào login path chính,
- login lưu `accessToken`, `role` và `userDetails` trong client state,
- việc chọn station và slot xảy ra trước khi booking được tạo,
- `BookingController.createBooking()` tạo `PENDING` booking và khóa công suất slot ngay lập tức,
- bước xác nhận booking là call riêng và trả về QR PNG,
- frontend session state lưu metadata của booking và QR image cho các trang phía sau.

## Nội dung

1. [Đăng ký, đăng nhập và trạng thái xác thực phía client](4.3.1-registration-login-and-client-state/)
2. [Khám phá trạm sạc và chọn slot](4.3.2-station-discovery-and-slot-selection/)
3. [Tạo booking và xác nhận booking](4.3.3-booking-creation-and-confirmation/)
4. [QR và xác thực booking](4.3.4-qr-and-booking-validation/)
