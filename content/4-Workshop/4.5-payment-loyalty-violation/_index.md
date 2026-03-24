---
title: "Invoice, Payment, Loyalty, and Violation Handling"
date: 2025-10-13
weight: 5
chapter: false
pre: " <b> 4.5. </b> "
---

## Flow overview

Once the charging transaction creates an `UNPAID` invoice, the system enters the payment, loyalty, and compliance phase. Here the frontend loads invoice data and payment choices, the backend applies discounts and processes payment, and the scheduler-driven compliance flow handles overdue bookings and driver violations.

## Key implementation points

- `Payment.jsx` loads invoice, payment, discount, and loyalty context before payment,
- `VnPayController.create()` can generate a VNPay URL or process a direct payment path,
- `InvoiceDiscountServiceImpl.apply()` runs before final payment settlement,
- `PaymentSuccess.jsx` resolves callback state after VNPay,
- successful payment finalizes loyalty,
- overdue `PENDING` and `CONFIRMED` bookings are canceled and may create violations and triplets.

## Content

1. [Invoice and Payment Entry](4.5.1-invoice-and-payment-entry/)
2. [VNPay Callback and Payment Result](4.5.2-vnpay-callback-and-payment-result/)
3. [Loyalty Finalization](4.5.3-loyalty-finalization/)
4. [Overdue Booking Penalties and Auto-Ban](4.5.4-overdue-penalties-and-auto-ban/)
5. [Payment and Compliance Validation](4.5.5-payment-compliance-validation/)
