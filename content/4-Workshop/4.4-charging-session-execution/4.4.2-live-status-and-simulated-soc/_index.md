---
title: "Live Charging View and Simulated SOC"
date: 2025-10-13
weight: 2
chapter: false
pre: " <b> 4.4.2. </b> "
---

## Frontend live view

`ChargingSession.jsx` polls the backend for session state, then simulates live SOC and energy locally in the browser. The page stores `session_{sessionId}_live_soc` in `sessionStorage` to keep the charging UI responsive between refreshes.

## Backend status behavior

The backend exposes charging status, but `getChargingStatus()` is still simulated rather than connected to real charger telemetry. This is an intentional implementation limitation, not a documentation omission.

## Important consequence

Because SOC is partly maintained in browser storage, the charging experience can differ across devices or user contexts. The system behaves like a full-stack demo flow with application-level state coordination rather than a hardware-authoritative charging platform.
