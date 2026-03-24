---
title: "Chốt điểm thưởng"
date: 2025-10-13
weight: 3
chapter: false
pre: " <b> 4.5.3. </b> "
---

## Discount chạy trước payment

Trước khi payment được chốt, `InvoiceDiscountServiceImpl.apply()` sẽ chạy trước. Điều đó có nghĩa giá trị invoice có thể thay đổi trước khi transaction cuối cùng được commit.

## Kết quả loyalty

Khi payment thành công, `LoyaltyFinalizeServiceImpl.finalizeOnPaymentSuccess()` áp dụng quy tắc đơn giản nhưng quan trọng:

- nếu người dùng đã tiêu điểm loyalty, số điểm của vehicle bị reset về `0`,
- ngược lại vehicle được cộng `+1` điểm.

Ở frontend, loyalty được hiểu theo quy tắc `1 point = 1%`, nên hành vi discount mà người dùng nhìn thấy và quy tắc finalize ở backend phải luôn đồng bộ.
