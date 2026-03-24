---
title: "Penalty cho booking quá hạn và auto-ban"
date: 2025-10-13
weight: 4
chapter: false
pre: " <b> 4.5.4. </b> "
---

## Compliance được kích hoạt bởi scheduler

Compliance flow thực tế không phải một trang policy chung chung. Nó là một đường enforcement được scheduler kích hoạt:

- `BookingScheduler.autoCancelOverdueBookings()` tìm các booking `PENDING` và `CONFIRMED` đã quá hạn,
- `BookingOverdueHandler.cancelAndCreateViolationTx()` hủy chúng,
- `ViolationServiceImpl.createViolation()` tính penalty bằng số phút đã giữ chỗ nhân với giá tariff,
- hệ thống lưu `DriverViolation`, gom vi phạm thành triplet, và auto-ban driver khi active violations đạt `3`.

## Mức độ hiển thị ở frontend

Frontend có các màn hình admin/staff liên quan thông qua API cho policy, triplet và report, nhưng việc tính penalty và ra quyết định ban thật sự nằm trong backend services và scheduler flows.
