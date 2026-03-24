---
title: "Workshop"
date: 2025-10-13
weight: 4
chapter: false
pre: " <b> 4. </b> "
---

# Workshop: EV Charging Station Management Full-Stack Flow

## Overview

This Workshop section documents the **actual end-to-end system flow** of the EV Charging Station Management System across the React frontend, the Spring Boot backend, and the supporting integrations around storage, notifications, payments, and maps. It is no longer an AWS-generic tutorial. It is a technical walkthrough of how the current product behaves from the moment a user signs in until a booking is charged, paid, and possibly penalized.

The content is organized around the implemented runtime sequence:

1. users register, verify OTP, and log in,
2. the frontend loads stations, slot availability, and vehicle context,
3. the backend creates and confirms bookings,
4. staff or driver-facing flows start and stop charging sessions,
5. invoice and payment logic settle the commercial flow,
6. loyalty and compliance logic finalize the operational outcome.

## Full-stack scope

- **Frontend**: React routing, protected routes, API wrappers, local state, Redux, `localStorage`, and `sessionStorage`
- **Backend**: Spring Boot monolith with controllers, services, repositories, schedulers, events, and async notifications
- **Integrations**: JWT auth, OTP verification, VNPay, SMTP email, AWS S3, AWS Location, and media/storage configuration such as Cloudinary when enabled

![EV Charging System Architecture](/images/4-workshop/ev-achi.png)

## Workshop objectives

* Explain the real driver journey from station discovery to payment success
* Map frontend user actions to backend controllers and service transactions
* Clarify important status transitions for bookings, sessions, invoices, loyalty, and violations
* Document implementation realities and known limitations honestly

## Content

1. [Introduction](4.1-introduction/)
2. [Prerequisites](4.2-prerequisites/)
3. [Authentication and Booking Flow](4.3-auth-booking-flow/)
4. [Charging Session Execution](4.4-charging-session-execution/)
5. [Invoice, Payment, Loyalty, and Violation Handling](4.5-payment-loyalty-violation/)
6. [Validation, Results, Limitations, and Cleanup](4.6-validation-results-cleanup/)
