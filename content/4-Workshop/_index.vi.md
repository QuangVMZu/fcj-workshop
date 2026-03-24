---
title: "Workshop"
date: 2025-10-13
weight: 4
chapter: false
pre: " <b> 4. </b> "
---

# Workshop: Luồng full-stack của hệ thống quản lý trạm sạc xe điện

## Tổng quan

Section Workshop này mô tả **luồng hệ thống end-to-end thực tế** của EV Charging Station Management System trên cả React frontend, Spring Boot backend, và các tích hợp xung quanh như storage, notification, payment, và bản đồ. Đây không còn là một tutorial AWS generic. Nó là tài liệu walkthrough kỹ thuật về cách sản phẩm hiện tại vận hành từ lúc người dùng đăng nhập cho đến khi booking được sạc, thanh toán, và trong một số trường hợp bị xử lý vi phạm.

Nội dung được tổ chức theo đúng trình tự runtime đang được triển khai:

1. người dùng đăng ký, xác thực OTP và đăng nhập,
2. frontend tải station, slot availability và vehicle context,
3. backend tạo và xác nhận booking,
4. staff hoặc driver-facing flow khởi động và dừng charging session,
5. invoice và payment logic chốt luồng thương mại,
6. loyalty và compliance logic hoàn tất kết quả vận hành.

## Phạm vi full-stack

- **Frontend**: React routing, protected routes, API wrappers, local state, Redux, `localStorage` và `sessionStorage`
- **Backend**: Spring Boot monolith với controllers, services, repositories, schedulers, events và async notifications
- **Tích hợp**: JWT auth, OTP verification, VNPay, SMTP email, AWS S3, AWS Location và cấu hình media/storage như Cloudinary khi được bật

![EV Charging System Architecture](/images/4-workshop/ev-achi.png)

## Mục tiêu workshop

* Giải thích hành trình người dùng thực tế từ lúc xem trạm sạc đến lúc thanh toán thành công
* Ánh xạ các thao tác của frontend vào controller và transaction của backend
* Làm rõ các trạng thái quan trọng của booking, session, invoice, loyalty và violation
* Ghi nhận trung thực các giới hạn hiện tại của implementation

## Nội dung

1. [Giới thiệu](4.1-introduction/)
2. [Điều kiện tiên quyết](4.2-prerequisites/)
3. [Luồng xác thực và đặt chỗ](4.3-auth-booking-flow/)
4. [Thực thi phiên sạc](4.4-charging-session-execution/)
5. [Hóa đơn, thanh toán, điểm thưởng và vi phạm](4.5-payment-loyalty-violation/)
6. [Xác thực, kết quả, giới hạn và tổng kết](4.6-validation-results-cleanup/)
