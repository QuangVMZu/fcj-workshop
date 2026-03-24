---
title: "Payment and Compliance Validation"
date: 2025-10-13
weight: 5
chapter: false
pre: " <b> 4.5.5. </b> "
---

## Validation checklist

Confirm that:

- invoice state, transaction state, and frontend payment result page remain consistent,
- the direct payment and VNPay paths both feed the same commercial outcome,
- discount application occurs before final settlement,
- loyalty changes match the point-usage rule,
- overdue bookings can produce violations and triplets,
- repeated violations can escalate into a driver ban.

## Known friction points

- invoice and VNPay state may become inconsistent,
- payment retry navigation is brittle on the frontend,
- response-shape handling is inconsistent in several client API flows.
