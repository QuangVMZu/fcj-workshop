---
title: "QR and Booking Validation"
date: 2025-10-13
weight: 4
chapter: false
pre: " <b> 4.3.4. </b> "
---

## What should be validated

After a successful booking cycle:

- the booking exists first as `PENDING` and later as `CONFIRMED`,
- slot capacity is already blocked while the booking is still pending,
- the confirmed booking has `BookingSlotLog` rebuilt,
- a QR PNG is returned and stored by the frontend as `qr_booking_{bookingId}`,
- the booking detail page can render the booking and operate on confirmation or cancellation correctly.

## Honest implementation risks

The current implementation also reveals two important risks:

- `PENDING` bookings already block capacity, which may reduce availability aggressively,
- slot reservation still carries race-condition risk if multiple requests compete before the database state settles.
