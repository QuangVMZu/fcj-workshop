---
title: "Callback VNPay và kết quả thanh toán"
date: 2025-10-13
weight: 2
chapter: false
pre: " <b> 4.5.2. </b> "
---

## Các nhánh của payment path

`VnPayController.create()` hỗ trợ hai hướng chính:

- tạo hosted VNPay URL thông qua `PaymentService.createVnPayPaymentUrl()`,
- hoặc xử lý một direct payment path qua tầng service.

## Xử lý callback

Khi người dùng quay về từ VNPay, `PaymentSuccess.jsx` xử lý callback state ở phía frontend. Ở backend, `VnPayController.handleReturn()` delegate sang `PaymentService.handleVnPayReturn()`, nơi việc kiểm tra chữ ký, tìm invoice, tìm transaction và cập nhật trạng thái được thực hiện.

## Vì sao bước này quan trọng

Đây là điểm giao nhau giữa UX thanh toán ở frontend và tính toàn vẹn payment ở backend. Nếu callback handling không nhất quán, trạng thái invoice, transaction và trạng thái thành công mà người dùng nhìn thấy có thể lệch nhau.
