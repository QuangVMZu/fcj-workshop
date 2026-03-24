---
title: "Tạo booking và xác nhận booking"
date: 2025-10-13
weight: 3
chapter: false
pre: " <b> 4.3.3. </b> "
---

## Hình dạng request từ frontend

`Booking.jsx` cuối cùng sẽ gửi `createBooking({ vehicleId, slotIds })`. Trang này cũng chuẩn bị sẵn client context như max charging power và vehicle được chọn để các màn hình phía sau tiếp tục sử dụng.

## Tạo booking ở backend

Luồng xử lý phía server là:

`BookingController.createBooking()` -> `BookingServiceImpl.createBooking()`

Implementation sau đó sẽ:

1. tùy chọn load vehicle,
2. load `SlotAvailability` theo id,
3. chỉ chấp nhận các dòng vẫn còn trạng thái `AVAILABLE`,
4. tạo `Booking(PENDING)`,
5. tạo các dòng `BookingSlot`,
6. ngay lập tức chuyển các slot đã chọn sang `BOOKED`.

## Bước xác nhận

`BookingDetail.jsx` sau đó tải lại booking và cho phép confirm hoặc cancel. Khi xác nhận, `confirmBooking()` sẽ:

- chuyển booking sang `CONFIRMED`,
- rebuild `BookingSlotLog`,
- tạo notification,
- trả về QR PNG.

Frontend lưu `bookingId`, `booking_{id}_maxPowerKW` và payload QR image để dùng ở các bước sau.
