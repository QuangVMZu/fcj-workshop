---
title: "Key Implementation Observations and Known Risks"
date: 2025-10-13
weight: 2
chapter: false
pre: " <b> 4.6.2. </b> "
---

## Backend-side observations

- secrets are committed in `application.properties`,
- authorization behavior is not fully consistent,
- booking reservation still carries race-condition risk,
- `PENDING` bookings already block capacity,
- overdue cancellation may fail to release booked slots completely,
- charging telemetry is simulated,
- invoice and VNPay state can drift,
- violation flow is not fully closed-loop,
- OAuth2 wiring is uneven,
- test coverage is very limited.

## Frontend-side observations

- UI authorization trusts client state and `?token=` behavior too much,
- stop-session behavior depends on shared `sessionStorage` state across devices,
- OTP resend is only simulated in the UI,
- payment retry navigation is broken,
- OAuth callback logic is duplicated,
- charging point status semantics are inconsistent,
- some UI code is brittle, including missing imports and inconsistent response-shape handling.

## Interpretation

These issues do not invalidate the Workshop. They define the real maturity level of the current implementation and should be documented explicitly rather than hidden.
