---
title: "Authentication and Booking Flow"
date: 2025-10-13
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

## Flow overview

The authentication and booking phase spans both frontend and backend. On the client side, the user signs in, browses stations, loads slot data, selects a vehicle, and submits a booking request. On the server side, the system authenticates the request, validates slot ownership, creates the booking, confirms it later, and returns a QR image for the next operational step.

## Key implementation points

- registration uses OTP verification before the main login path,
- login persists `accessToken`, `role`, and `userDetails` in client state,
- station and slot selection happen before booking creation,
- `BookingController.createBooking()` creates a `PENDING` booking and blocks slot capacity immediately,
- confirmation is separate and returns a QR PNG,
- frontend session state stores booking metadata and the QR image for later pages.

## Content

1. [Registration, Login, and Client Auth State](4.3.1-registration-login-and-client-state/)
2. [Station Discovery and Slot Selection](4.3.2-station-discovery-and-slot-selection/)
3. [Booking Creation and Confirmation](4.3.3-booking-creation-and-confirmation/)
4. [QR and Booking Validation](4.3.4-qr-and-booking-validation/)
