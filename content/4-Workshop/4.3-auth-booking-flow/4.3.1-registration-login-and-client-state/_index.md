---
title: "Registration, Login, and Client Auth State"
date: 2025-10-13
weight: 1
chapter: false
pre: " <b> 4.3.1. </b> "
---

## Frontend auth entry points

The frontend begins with:

- `Register.jsx` calling `registerApi()`,
- `Verify.jsx` calling `verifyOtp()`,
- `Login.jsx` calling `useLogin()` and then `loginApi()` against `POST /api/users/login`.

On success, the client stores `accessToken`, `role`, and `userDetails` in `localStorage` and Redux. If the authenticated role is `STAFF`, the frontend also resolves station assignment data before routing the user into staff-scoped views.

## Backend auth path

The backend path is:

`UsersController.login()` -> `UserServiceImpl.authenticate()` -> `TokenService.generateToken()`

After that, `JwtAuthFilter` protects subsequent requests.

## Client-state implications

This flow is important because later booking, charging, payment, and notification screens all depend on previously stored client state. The current implementation also reflects some trust in client-held auth state, which becomes relevant in the limitations section.
