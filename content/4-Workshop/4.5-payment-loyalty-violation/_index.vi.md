---
title: "Hóa đơn, thanh toán, điểm thưởng và vi phạm"
date: 2025-10-13
weight: 5
chapter: false
pre: " <b> 4.5. </b> "
---

## Tổng quan luồng

Ngay khi charging transaction tạo ra `UNPAID` invoice, hệ thống chuyển sang giai đoạn payment, loyalty và compliance. Ở đây frontend tải invoice data và các lựa chọn thanh toán, backend áp dụng discount rồi xử lý payment, còn scheduler-driven compliance flow chịu trách nhiệm hủy các booking quá hạn và tạo violation cho driver.

## Các điểm implementation quan trọng

- `Payment.jsx` tải invoice, payment, discount và loyalty context trước khi bắt đầu thanh toán,
- `VnPayController.create()` có thể sinh VNPay URL hoặc xử lý direct payment,
- `InvoiceDiscountServiceImpl.apply()` chạy trước khi payment được chốt,
- `PaymentSuccess.jsx` xử lý callback state sau khi VNPay quay về,
- payment thành công sẽ finalize loyalty,
- các booking `PENDING` và `CONFIRMED` quá hạn có thể bị hủy và tạo violation hoặc triplet.

## Nội dung

1. [Điểm vào của invoice và payment](4.5.1-invoice-and-payment-entry/)
2. [Callback VNPay và kết quả thanh toán](4.5.2-vnpay-callback-and-payment-result/)
3. [Chốt điểm thưởng](4.5.3-loyalty-finalization/)
4. [Penalty cho booking quá hạn và auto-ban](4.5.4-overdue-penalties-and-auto-ban/)
5. [Xác thực payment và compliance](4.5.5-payment-compliance-validation/)
