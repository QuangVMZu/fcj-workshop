---
title: "Giới thiệu"
date: 2025-10-13
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

## Mục đích của workshop

Workshop này giải thích cách hệ thống EV Charging vận hành như một **ứng dụng full-stack**, chứ không chỉ như một tập hợp endpoint backend. Hành trình của driver bắt đầu từ React frontend, đi qua API wrappers và protected routes, chạm vào các controller và service của Spring Boot, rồi quay trở lại giao diện qua luồng payment, notification và dashboard.

## Tổng quan hệ thống

Ở mức khái quát, implementation hiện tại bao gồm:

- một ứng dụng React dành cho driver, staff và administrator,
- một Spring Boot monolith chịu trách nhiệm cho các transaction nghiệp vụ chính,
- dữ liệu quan hệ cho station, slot, booking, charging session, invoice, loyalty và violation,
- các tích hợp cho OTP, JWT, VNPay, SMTP, frontend assets host trên AWS, AWS Location, và tùy chọn media storage adapter.

![EV Charging System Architecture](/images/2-Proposal/aws-ev-architecture.png)

## Vì sao thứ tự luồng nghiệp vụ quan trọng

Các module liên kết chặt với nhau. Booking phải giữ slot trước khi charging có thể bắt đầu. Charging phải hoàn tất trước khi invoice xuất hiện. Payment finalize mới cập nhật loyalty. Việc dọn booking quá hạn có thể tạo violation và cuối cùng ban driver. Vì vậy Workshop được tổ chức theo vòng đời hệ thống thực tế thay vì tách rời thành các dịch vụ cloud rời rạc.
