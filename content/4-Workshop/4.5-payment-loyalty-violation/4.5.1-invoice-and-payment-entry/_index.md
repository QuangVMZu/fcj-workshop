---
title: "Invoice and Payment Entry"
date: 2025-10-13
weight: 1
chapter: false
pre: " <b> 4.5.1. </b> "
---

## Invoice entry

The payment flow starts from the invoice created by `ChargingSessionTxHandler.stopSessionInternalTx()`. That invoice is the commercial source of truth for the next step.

## Frontend payment screen

`Payment.jsx` loads:

- invoice detail,
- payment methods,
- discount information,
- loyalty context.

The frontend also hides `VNPAY` and `EWALLET` options when the payable total is below `10000`, so the UI already shapes the payment path before the backend executes it.

## Backend payment entry

On the server side, the payment logic branches from `VnPayController.create()` and related payment services.
