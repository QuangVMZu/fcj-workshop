---
title: "Danh sách xác thực end-to-end"
date: 2025-10-13
weight: 1
chapter: false
pre: " <b> 4.6.1. </b> "
---

## Checklist của hành trình đầy đủ

Một lượt test end-to-end của implementation hiện tại nên xác nhận chuỗi hành vi sau:

1. đăng ký và xác thực OTP,
2. đăng nhập và lưu client auth state,
3. duyệt station và chọn slot hợp lệ,
4. tạo booking và xác nhận booking,
5. lưu và tái sử dụng booking QR,
6. bắt đầu charging session,
7. quan sát các cập nhật SOC dạng mô phỏng,
8. dừng session và tạo invoice,
9. thanh toán bằng direct payment hoặc VNPay,
10. chốt loyalty,
11. quan sát hành vi violation cho booking quá hạn nếu có.

## Bằng chứng cần kiểm tra

- API responses,
- trạng thái lưu trữ ở frontend,
- các lần chuyển trạng thái trong database,
- notifications,
- invoices và transactions,
- loyalty và violation records.
