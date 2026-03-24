---
title: "VNPay Callback and Payment Result"
date: 2025-10-13
weight: 2
chapter: false
pre: " <b> 4.5.2. </b> "
---

## Payment path options

`VnPayController.create()` supports two main directions:

- create a hosted VNPay URL through `PaymentService.createVnPayPaymentUrl()`,
- or process a more direct payment path through the service layer.

## Callback resolution

When the user returns from VNPay, `PaymentSuccess.jsx` resolves the callback state on the frontend. On the backend, `VnPayController.handleReturn()` delegates to `PaymentService.handleVnPayReturn()`, where signature validation, invoice lookup, transaction lookup, and status updates take place.

## Why this step matters

This is the point where the frontend payment UX and backend payment integrity meet. If callback processing is inconsistent, invoice state, transaction state, and user-visible payment success can drift apart.
