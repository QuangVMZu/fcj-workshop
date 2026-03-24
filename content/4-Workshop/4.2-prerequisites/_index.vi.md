---
title: "Điều kiện tiên quyết"
date: 2025-10-13
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

## Điều kiện phía frontend

Phía frontend giả định có một ứng dụng React với:

- routing được khai báo qua entry point chính và app router,
- protected-route handling cho các trang có phân quyền,
- API wrappers tự gắn bearer token và xử lý phản hồi `401`,
- cơ chế lưu trạng thái qua Redux, `localStorage` và `sessionStorage`,
- các module cho auth, station browsing, booking, charging, payment, notifications và staff/admin operations.

## Điều kiện phía backend

Phía backend giả định có một Spring Boot monolith với:

- REST controllers cho auth, booking, charging session, invoice, VNPay, loyalty, notification, report và policy,
- tầng service điều phối transaction nghiệp vụ,
- JPA entities và repositories cho booking, slot, session, invoice, loyalty và violation,
- scheduler và async events cho overdue cleanup, slot generation, auto-stop, notifications và email fan-out.

## Điều kiện về hạ tầng và tích hợp

Luồng end-to-end còn phụ thuộc vào:

- cơ sở dữ liệu quan hệ đã có users, stations, tariffs, slots, vehicles và policy data,
- JWT security cùng phần OAuth2 đang được nối ở mức chưa đồng đều,
- OTP verification và SMTP delivery,
- cấu hình VNPay cho hosted payment và callback handling,
- các tích hợp cho frontend assets và media như AWS S3, AWS Location và Cloudinary nếu môi trường đang bật.

## Giả định khởi tạo quan trọng

Trước khi workflow có ý nghĩa, hệ thống phải có sẵn dữ liệu station, tariff, slot generation, active vehicles và ít nhất một driver account có thể sử dụng. Nếu thiếu các điều kiện này, hành trình frontend và booking logic của backend sẽ không thể chạy một cách nhất quán.
