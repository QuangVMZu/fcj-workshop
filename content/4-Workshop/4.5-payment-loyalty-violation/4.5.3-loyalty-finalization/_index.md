---
title: "Loyalty Finalization"
date: 2025-10-13
weight: 3
chapter: false
pre: " <b> 4.5.3. </b> "
---

## Discount before payment

Before payment is finalized, `InvoiceDiscountServiceImpl.apply()` runs first. This means invoice value can change before the final transaction is committed.

## Loyalty outcome

On successful payment, `LoyaltyFinalizeServiceImpl.finalizeOnPaymentSuccess()` applies a simple but important rule:

- if loyalty points were consumed, the vehicle points reset to `0`,
- otherwise the vehicle receives `+1` point.

On the frontend, loyalty is treated as `1 point = 1%`, so the user-visible discount behavior and the backend finalization rule must stay aligned.
