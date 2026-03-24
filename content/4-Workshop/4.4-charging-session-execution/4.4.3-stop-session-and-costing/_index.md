---
title: "Stop Flow, Auto-Stop, and Costing"
date: 2025-10-13
weight: 3
chapter: false
pre: " <b> 4.4.3. </b> "
---

## Stop entry points

The stop path can be triggered by:

- driver stop endpoints,
- staff stop endpoints,
- the automatic stop scheduled at booking end.

The frontend charging views also try to reuse browser-stored SOC when stop actions are performed.

## Transaction boundary

All stop paths converge into `ChargingSessionTxHandler.stopSessionInternalTx()`.

That transactional handler computes:

- duration,
- charging point and connector context,
- estimated final SOC,
- energy,
- tariff and cost,
- overstay minutes beyond the estimated time required to reach 100 percent SOC.

When it succeeds, the session is completed, the booking is completed, a notification is created, and an `Invoice(UNPAID)` is produced for the payment flow.
