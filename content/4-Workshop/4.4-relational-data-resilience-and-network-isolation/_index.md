---
title: "Relational Data Resilience and Network Isolation"
date: 2026-04-04
weight: 4
chapter: false
pre: " <b> 4.4. </b> "
---

# Relational Data Resilience and Network Isolation

## Overview

- This section describes how the platform stores persistent application data while keeping the database tier isolated from public access.
- Its purpose is to provide durable relational storage and support availability in case of database or infrastructure failure.
- It fits behind the application layer as the system of record for the workload.

## Components

### Amazon RDS
![Amazon RDS](/images/4-Workshop/rds-overview.png)
- Managed relational database service for the application data tier.
- Handles provisioning, backups, patching, and failover.

### Primary Database Instance
![RDS Primary Instance](/images/4-Workshop/rds-primary.png)
- Handles the main transactional workload.
- Processes read and write operations from the application.

### Standby Database Instance
- Maintains a synchronized replica of the primary instance.
- Automatically takes over during failover scenarios.

### No Public Access Boundary
- Ensures the database is not publicly accessible.
- Access is restricted to internal resources within the VPC.

### EC2 to RDS Connectivity
- Application servers connect to the database over private networking.
- Typically uses security groups and internal DNS endpoints.

## Flow Description

1. EC2 instances send database reads and writes to the primary RDS instance.
2. The RDS service maintains synchronization between the primary and standby instances.
3. If the primary database becomes unavailable, the standby instance likely supports failover continuity.
4. Database access remains internal to the application environment and is not exposed directly to internet users.

## Key Logic / Rules

- The primary database is the main endpoint for normal application transactions.
- The standby database improves resilience but is not shown as a user-facing endpoint.
- Network isolation is a core rule of the design, as the diagram explicitly marks the data tier with no public access.
- Keeping the data tier private reduces attack surface and limits unintended connectivity paths.
- Application-layer control is required for all database interactions.

## Notes / Assumptions

- The primary and standby layout strongly suggests a Multi-AZ or equivalent high-availability configuration, but the exact mode is not labeled.
- Backup retention, point-in-time recovery, and encryption settings are not visible in the diagram.
- No read replica, cache, or analytics database is shown, so those patterns are not assumed here.
