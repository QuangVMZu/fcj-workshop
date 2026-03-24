---
title: "Booking Creation and Confirmation"
date: 2025-10-13
weight: 3
chapter: false
pre: " <b> 4.3.3. </b> "
---

## Frontend request shape

`Booking.jsx` eventually submits `createBooking({ vehicleId, slotIds })`. The page also prepares client context such as max charging power and the selected vehicle for later screens.

## Backend booking creation

The server-side path is:

`BookingController.createBooking()` -> `BookingServiceImpl.createBooking()`

The implementation then:

1. optionally loads the vehicle,
2. loads `SlotAvailability` by id,
3. accepts only rows still marked `AVAILABLE`,
4. creates `Booking(PENDING)`,
5. creates `BookingSlot` rows,
6. immediately flips the selected slots to `BOOKED`.

## Confirmation step

`BookingDetail.jsx` then loads the booking and can confirm or cancel it. On confirmation, `confirmBooking()`:

- changes the booking to `CONFIRMED`,
- rebuilds `BookingSlotLog`,
- creates a notification,
- returns a QR PNG.

The frontend stores `bookingId`, `booking_{id}_maxPowerKW`, and the QR image payload for later use.
