---
title: "Overdue Booking Penalties and Auto-Ban"
date: 2025-10-13
weight: 4
chapter: false
pre: " <b> 4.5.4. </b> "
---

## Scheduler-driven compliance

The real compliance flow is not a generic policy review. It is a scheduler-backed enforcement path:

- `BookingScheduler.autoCancelOverdueBookings()` finds overdue `PENDING` and `CONFIRMED` bookings,
- `BookingOverdueHandler.cancelAndCreateViolationTx()` cancels them,
- `ViolationServiceImpl.createViolation()` calculates the penalty from reserved minutes multiplied by tariff price,
- the system saves `DriverViolation`, groups violations into triplets, and auto-bans the driver once active violations reach `3`.

## Frontend visibility

The frontend exposes related admin/staff views through APIs for policies, triplets, and reports, but the actual penalty calculation and ban decision live in backend services and scheduler flows.
