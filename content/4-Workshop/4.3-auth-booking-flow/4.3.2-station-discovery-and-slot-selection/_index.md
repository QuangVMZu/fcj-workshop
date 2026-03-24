---
title: "Station Discovery and Slot Selection"
date: 2025-10-13
weight: 2
chapter: false
pre: " <b> 4.3.2. </b> "
---

## Frontend browsing flow

The driver journey then moves through the station pages:

- `Stations.jsx` loads stations using geolocation,
- `StationDetail.jsx` loads station detail, charging points, connector types, and vehicles,
- `Booking.jsx` loads slot availability and slot templates before the user can reserve.

## Client-side booking rules

Before calling the backend, the frontend already enforces several business rules:

- only same-day future slots can be chosen,
- a maximum of three slots may be selected,
- selected slots must be consecutive,
- only `ACTIVE` vehicles are selectable,
- connector compatibility is checked using connector text or code.

This means the booking flow is validated twice: once in the UI and once again in the backend services.
