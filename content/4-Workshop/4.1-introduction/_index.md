---
title: "Introduction"
date: 2025-10-13
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

## Workshop purpose

This workshop explains how the EV Charging system behaves as a **full-stack application**, not just as a backend API catalog. The driver journey starts in the React frontend, travels through API wrappers and protected routes, reaches Spring Boot controllers and services, and then returns through payment, notification, and dashboard flows.

## System overview

At a high level, the current implementation contains:

- a React application for drivers, staff, and administrators,
- a Spring Boot monolith that owns the main business transactions,
- relational data for stations, slots, bookings, charging sessions, invoices, loyalty, and violations,
- integrations for OTP, JWT, VNPay, SMTP, AWS-hosted frontend assets, AWS Location, and optional media storage adapters.

![EV Charging System Architecture](/images/2-Proposal/aws-ev-architecture.png)

## Why the flow order matters

The modules are tightly connected. A booking must reserve slots before charging can start. Charging must complete before an invoice exists. Payment finalization updates loyalty. Overdue booking cleanup may create violations and eventually ban the driver. Because of that coupling, the Workshop is organized around the real lifecycle rather than around isolated cloud services.
