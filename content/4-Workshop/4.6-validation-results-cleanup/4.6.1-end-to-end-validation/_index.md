---
title: "End-to-End Validation Checklist"
date: 2025-10-13
weight: 1
chapter: false
pre: " <b> 4.6.1. </b> "
---

## Full journey checklist

An end-to-end test of the implemented system should verify this full sequence:

1. register and verify OTP,
2. log in and persist client auth state,
3. browse stations and select valid slots,
4. create a booking and confirm it,
5. store and reuse the booking QR,
6. start a charging session,
7. view simulated SOC updates,
8. stop the session and create an invoice,
9. settle payment through direct payment or VNPay,
10. finalize loyalty,
11. observe overdue violation behavior when applicable.

## Evidence to inspect

- API responses,
- frontend storage state,
- database status transitions,
- notifications,
- invoices and transactions,
- loyalty and violation records.
