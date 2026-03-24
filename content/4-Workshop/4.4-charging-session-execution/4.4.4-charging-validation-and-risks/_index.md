---
title: "Charging Validation and Risks"
date: 2025-10-13
weight: 4
chapter: false
pre: " <b> 4.4.4. </b> "
---

## Validation checklist

Confirm the following after a charging run:

- only confirmed bookings can start a session,
- the booking becomes `BOOKED` at session start,
- the session remains `IN_PROGRESS` until stop or auto-stop,
- stopping the session creates an `UNPAID` invoice,
- the frontend charging view reflects the session progression coherently.

## Known charging-specific risks

- charging telemetry is simulated,
- the UI does not auto-stop itself at 100 percent SOC,
- staff stop behavior depends on browser-stored SOC that may not exist on another device,
- charging point status semantics are not perfectly consistent across the frontend and backend layers.
