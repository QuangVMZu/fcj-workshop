---
title: "Prerequisites"
date: 2025-10-13
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

## Frontend prerequisites

The frontend side assumes a React application with:

- routing defined through the main application entry and app router,
- protected-route handling for role-sensitive pages,
- API wrappers that inject bearer tokens and react to `401` responses,
- client persistence through Redux, `localStorage`, and `sessionStorage`,
- modules for auth, station browsing, booking, charging, payment, notifications, and staff/admin operations.

## Backend prerequisites

The backend side assumes a Spring Boot monolith with:

- REST controllers for auth, booking, charging session, invoice, VNPay, loyalty, notification, report, and policy flows,
- service-layer orchestration for transactional business logic,
- JPA entities and repositories for booking, slot, session, invoice, loyalty, and violation data,
- schedulers and async events for overdue cleanup, slot generation, auto-stop, notifications, and email fan-out.

## Infrastructure and integration prerequisites

The end-to-end flow also depends on:

- a relational database containing users, stations, tariffs, slots, vehicles, and policy data,
- JWT security plus partially wired OAuth2-related behavior,
- OTP verification and SMTP delivery,
- VNPay configuration for hosted payment and callback handling,
- frontend asset and file integrations such as AWS S3, AWS Location, and Cloudinary where the environment enables them.

## Important setup assumptions

Before the workflow is meaningful, the system should already have station data, tariff data, slot generation, active vehicles, and at least one usable driver account. Without those prerequisites, the frontend journey and the backend booking logic cannot be exercised coherently.
