---
title: "QR và xác thực booking"
date: 2025-10-13
weight: 4
chapter: false
pre: " <b> 4.3.4. </b> "
---

## Những gì cần được xác thực

Sau một chu trình booking thành công:

- booking trước hết tồn tại ở trạng thái `PENDING`, sau đó mới sang `CONFIRMED`,
- công suất slot đã bị khóa ngay cả khi booking vẫn đang pending,
- booking đã confirm sẽ có `BookingSlotLog` được rebuild,
- QR PNG được trả về và được frontend lưu thành `qr_booking_{bookingId}`,
- trang booking detail có thể render đúng booking và thao tác confirm hoặc cancel một cách chính xác.

## Các rủi ro implementation cần nói thật

Implementation hiện tại cũng bộc lộ hai rủi ro quan trọng:

- booking ở trạng thái `PENDING` đã chặn công suất, nên có thể làm availability giảm sớm,
- việc giữ slot vẫn có nguy cơ race condition nếu nhiều request cạnh tranh trước khi trạng thái database ổn định hoàn toàn.
