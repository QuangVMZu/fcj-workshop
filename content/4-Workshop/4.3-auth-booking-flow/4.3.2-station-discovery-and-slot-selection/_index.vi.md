---
title: "Khám phá trạm sạc và chọn slot"
date: 2025-10-13
weight: 2
chapter: false
pre: " <b> 4.3.2. </b> "
---

## Luồng duyệt trạm ở frontend

Hành trình của driver tiếp tục qua các màn hình trạm sạc:

- `Stations.jsx` tải danh sách station dựa trên geolocation,
- `StationDetail.jsx` tải thông tin station, charging point, connector type và vehicles,
- `Booking.jsx` tải slot availability và slot template trước khi người dùng được phép đặt chỗ.

## Các quy tắc booking ở phía client

Trước khi gọi backend, frontend đã áp dụng một số business rule:

- chỉ được chọn slot tương lai trong cùng ngày,
- tối đa chỉ được chọn ba slot,
- các slot được chọn phải liên tiếp,
- chỉ vehicle ở trạng thái `ACTIVE` mới được chọn,
- tính tương thích của connector được kiểm tra bằng text hoặc code của connector.

Điều này có nghĩa luồng booking được kiểm tra hai lần: một lần ở UI và một lần nữa trong service của backend.
