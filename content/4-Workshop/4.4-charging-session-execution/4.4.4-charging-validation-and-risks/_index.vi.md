---
title: "Xác thực luồng charging và các rủi ro"
date: 2025-10-13
weight: 4
chapter: false
pre: " <b> 4.4.4. </b> "
---

## Danh sách xác thực

Sau một lượt charging, cần xác nhận:

- chỉ booking đã `CONFIRMED` mới được phép bắt đầu session,
- booking chuyển sang `BOOKED` tại thời điểm session bắt đầu,
- session giữ trạng thái `IN_PROGRESS` cho đến khi stop hoặc auto-stop,
- khi dừng session, hệ thống sinh `Invoice(UNPAID)`,
- charging view ở frontend phản ánh được tiến trình session một cách nhất quán.

## Các rủi ro đặc thù của charging

- charging telemetry hiện là mô phỏng,
- UI không tự dừng ở ngưỡng 100 phần trăm SOC,
- luồng staff stop phụ thuộc vào SOC lưu trong browser nên có thể không dùng được ở thiết bị khác,
- ý nghĩa trạng thái của charging point chưa hoàn toàn nhất quán giữa frontend và backend.
