---
title: "Proposal"
date: 2026-03-24
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# EV Charging Station Management System

# A Full-Stack Platform for Charging Operations and Digital Payments

---

## 1. Executive Summary

The **EV Charging Station Management System** is a full-stack platform designed to digitize, standardize, and optimize the entire operation of EV charging services in the context of rapidly growing electric vehicle adoption.

The system provides an end-to-end integrated solution that enables users to locate charging stations, book time slots, monitor charging sessions in real time, make payments, and receive notifications seamlessly. At the same time, it supports operators in managing charging resources, monitoring violations, tracking performance, and ensuring regulatory compliance.

From a technical perspective, the system is built on a modern full-stack architecture. The frontend serves as the role-based user interface (driver, staff, admin), while the backend handles all business logic, ensuring data consistency and integrity.

The system is deployed on AWS with the following core components:

- Amazon S3 + CloudFront (with WAF) for fast and secure frontend delivery
- Application Load Balancer + EC2 Auto Scaling for a scalable and highly available backend
- Amazon RDS SQL Server (Single-AZ) for data storage
- NAT Gateway for outbound internet access from private subnets
- Amazon SES for email notifications
- AWS Location Service for maps and geolocation
- AWS Systems Manager (SSM) + IAM for secure access without public IP

This solution helps operators:

- Optimize charging station utilization
- Reduce operational costs through automation
- Improve user experience
- Standardize systems and reduce technical debt
- Scale efficiently as demand grows

---

## 2. Problem Statement

### Current Issues

- No real-time slot visibility
- Inefficient booking and charging workflow
- Slow invoicing and payment processing
- Difficulty in violation monitoring
- Lack of synchronization between frontend and backend

### Frontend Issues

- Scattered business logic
- Non-standardized APIs
- Poor performance

---

### Solution

- Centralize all business logic in the backend
- Role-based and domain-based frontend architecture
- Standardized APIs using JWT
- Domain-driven modularization

### Key Features

- Authentication (JWT + Google OAuth)
- Booking / Charging / Payment
- Email notifications (SES)
- Map integration (AWS Location Service)

---

### Benefits

- **Improved performance**
- **Better user experience (UX)**
- **Scalability**
- **Higher system stability**

---

## 3. Solution Architecture

- **Edge Layer:** Route 53, CloudFront, WAF
- **Application Layer:** ALB + EC2 Auto Scaling (Multi-AZ)
- **Data Layer:** RDS SQL Server (private, Single-AZ)
- **Support Services:** IAM, SSM, Secrets Manager, CloudWatch

### Main Flow

1. User → Route 53  
2. CloudFront → S3  
3. Frontend → ALB  
4. ALB → EC2  
5. EC2 → RDS  
6. EC2 → SES / NAT Gateway  

### Key Characteristics

- EC2 instances run in private subnets (no public IP)
- Outbound traffic goes through NAT Gateway
- Secure access via SSM (no SSH required)
- Credentials managed by Secrets Manager

![EV Charging System Architecture](/images/2-Proposal/AWS-architecture.png)

---

## 4. Architecture Justification

### Design Rationale

**1. CloudFront + S3**

- Reduce latency
- Accelerate frontend delivery
- Offload backend traffic

**2. ALB + Auto Scaling Group**

- High availability
- Automatic scaling
- Stateless backend design

**3. EC2**

- Well-suited for Spring Boot
- High flexibility and control

**4. RDS Single-AZ**

- Cost-efficient
- Easy to upgrade later

**5. NAT Gateway**

- Secure outbound access
- No need for public IPs

**6. SSM + IAM**

- No SSH required
- Secure access management

**7. Secrets Manager**

- Secure credential storage

**8. CloudWatch + CloudTrail**

- Monitoring and auditing

---

### Trade-offs

- No Multi-AZ database (lower availability)
- EC2 requires manual management
- NAT Gateway cost is relatively high

---

## 5. Technical Implementation

- React + Axios + JWT
- Spring Boot
- RDS SQL Server
- AWS Cloud
- VNPay integration
- Amazon SES

---

## 6. Timeline

- Week 7–8: System setup and architecture design
- Week 8: Booking & charging session implementation
- Week 9: Payment and map integration
- Week 10: Loyalty & compliance features
- Week 11: Testing & optimization
- Week 12: Deployment

---

## 7. Budget Estimation

### Includes

- S3 + CloudFront
- EC2 (Auto Scaling)
- RDS SQL Server
- Route 53
- SES
- NAT Gateway

### Assumptions

- Single region
- 2 EC2 instances
- 1 RDS instance
- 1 NAT Gateway

### Estimated Cost

| Service     | Cost (USD/month) |
|------------|------------------|
| EC2         | ~30              |
| RDS         | ~26              |
| NAT Gateway | ~32–35           |
| Route 53    | ~0.5             |
| SES         | ~1               |
| **Total**   | **~90 USD**      |

---

## 8. Risk Assessment

- Non-standardized APIs
- Frontend performance limitations

---

## 9. Expected Outcomes

- Scalable system
- Stable performance
- Production-ready platform