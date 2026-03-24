---
title: "Starting a Session"
date: 2025-10-13
weight: 1
chapter: false
pre: " <b> 4.4.1. </b> "
---

## Frontend entry points

The actual session start is exposed through staff-oriented flows such as `SessionChargingCreate.jsx` and `InstantCharging.jsx`. Instant charging may use `vehicleId: null`, while a normal session is still anchored to a confirmed booking.

## Backend start flow

The backend path is:

`ChargingSessionController.startChargingSession()`

The implementation accepts only:

- bookings already in `CONFIRMED` status,
- requests inside the scheduled reservation window.

When the request is accepted, the system creates `ChargingSession(IN_PROGRESS)`, randomizes the initial SOC, caches SOC state, changes the booking status to `BOOKED`, and schedules an automatic stop at the booking end time.
