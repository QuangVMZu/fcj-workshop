---
title: "Bắt đầu phiên sạc"
date: 2025-10-13
weight: 1
chapter: false
pre: " <b> 4.4.1. </b> "
---

## Điểm vào ở frontend

Việc bắt đầu session được mở ra thông qua các flow dành cho staff như `SessionChargingCreate.jsx` và `InstantCharging.jsx`. Instant charging có thể dùng `vehicleId: null`, còn session tiêu chuẩn vẫn gắn với một booking đã được xác nhận.

## Backend start flow

Luồng phía backend là:

`ChargingSessionController.startChargingSession()`

Implementation chỉ chấp nhận:

- booking đã ở trạng thái `CONFIRMED`,
- request còn nằm trong khung giờ reservation hợp lệ.

Khi request hợp lệ, hệ thống tạo `ChargingSession(IN_PROGRESS)`, random giá trị SOC ban đầu, cache trạng thái SOC, đổi booking sang `BOOKED`, và lên lịch auto-stop tại thời điểm kết thúc booking.
