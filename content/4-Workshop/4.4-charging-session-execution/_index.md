---
title: "Charging Session Execution"
date: 2025-10-13
weight: 4
chapter: false
pre: " <b> 4.4. </b> "
---

## Flow overview

After booking confirmation, the system moves into charging execution. This part combines staff-facing session start screens, driver-facing charging views, backend scheduling, and the transaction boundary that completes the session and produces the invoice.

## Key implementation points

- staff can start sessions through session-management screens, while charging views are rendered on the client side,
- the backend only starts a session from a valid `CONFIRMED` booking inside the scheduled window,
- SOC and live charging progress are simulated rather than read from hardware,
- session progress is reflected both in backend status and in browser-stored temporary state,
- stop logic calculates cost and creates the invoice that payment uses next.

## Content

1. [Starting a Session](4.4.1-start-session/)
2. [Live Charging View and Simulated SOC](4.4.2-live-status-and-simulated-soc/)
3. [Stop Flow, Auto-Stop, and Costing](4.4.3-stop-session-and-costing/)
4. [Charging Validation and Risks](4.4.4-charging-validation-and-risks/)
