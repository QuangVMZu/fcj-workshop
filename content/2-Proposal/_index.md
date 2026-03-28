---
title: "Proposal"
date: 2026-03-24
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# EV Charging Station Management System

# Smart Full-Stack Platform for EV Charging Operations and Digital Payments

### 1. Executive Summary

The EV Charging Station Management System is a full-stack platform designed to support the complete operational lifecycle of electric vehicle charging services, covering both **backend operations** and **frontend user interaction layers**.

The system manages core processes such as station discovery, time-slot scheduling, booking confirmation, charging session tracking, invoice generation, payment processing, and driver compliance handling within a unified architecture.

On the backend side, the platform is built using Spring Boot and integrated with cloud-based infrastructure. On the frontend side, it provides a role-based web application supporting drivers, staff, and administrators with tailored interfaces and workflows.

The frontend communicates with the backend through a standardized API layer using Axios with JWT authentication, covering multiple domains such as authentication, station discovery, booking, charging session, invoice/payment, notification, and reporting.

The platform integrates with VNPay for payments, AWS S3 and CloudFront for frontend hosting and delivery, Amazon RDS for SQL Server for data persistence, AWS Map for geolocation services, and SMTP-based services for notifications.

This solution enables EV charging operators to digitize operations, improve efficiency, standardize workflows, and deliver a seamless end-to-end user experience.

---

### 2. Problem Statement

#### What's the Problem?

As EV adoption grows, operators face increasing complexity not only in backend operations but also in frontend user flows and system integration.

Key challenges include:

- Limited visibility into real-time slot availability
- Inefficient booking and session workflows
- Delays in billing and payment reconciliation
- Weak enforcement of operational policies
- Fragmented frontend-backend integration
- Inconsistent user experience across roles
- Heavy dependency on browser storage for workflow handling
- Lack of standardized API integration across frontend modules

On the frontend side, additional technical issues arise:

- Business logic scattered across page components
- Inconsistent API calling patterns
- Lack of centralized server-state management
- Performance bottlenecks due to polling and large bundles
- Navigation inconsistencies and hard page reloads

Without a unified full-stack architecture, the system becomes difficult to scale, maintain, and extend.

---

#### The Solution

The proposed solution is a **full-stack EV charging management platform** that standardizes both backend and frontend architecture.

It provides:

- Centralized backend for operational logic and data processing
- Role-based frontend architecture for driver, staff, and admin workflows
- Unified API communication layer between frontend and backend
- Structured domain-based modules (auth, booking, charging, payment, etc.)
- Reduced reliance on browser storage through better state management
- Improved performance through modular frontend architecture

Key capabilities include:

- User authentication and role-based authorization
- Booking and slot reservation workflows
- Charging session lifecycle management
- Invoice generation and billing
- VNPay payment integration
- Loyalty and compliance management
- Cloud-based frontend delivery via S3 + CloudFront
- API routing via Nginx
- Map integration via AWS Map
- Automated scheduling and background processing

---

#### Benefits and Return on Investment

The full-stack platform delivers both technical and business value:

- **Higher station utilization** through optimized booking workflows
- **Improved UX** with consistent frontend behavior and faster interactions
- **Reduced technical debt** through standardized architecture
- **Operational efficiency** via automation and integration
- **Scalability** across multiple stations and increasing users
- **Better maintainability** with domain-based frontend and backend modules
- **Improved reliability** by reducing client-side dependency on session/local storage

---

### 3. Solution Architecture

The platform follows a **full-stack layered architecture** with clear separation between frontend, backend, and data layers.

End users access the frontend via CloudFront, which serves static assets from Amazon S3. The frontend communicates with backend APIs through a centralized Axios client with JWT authentication. API requests are routed via Nginx to the Spring Boot backend. Data is stored in Amazon RDS for SQL Server.

![EV Charging System Architecture](/images/2-Proposal/aws-ev-architecture.png)

| Component | Service / Technology |
| --------------------- | -------------------------- |
| Frontend Framework | React (Vite) |
| Frontend Hosting | Amazon S3 |
| CDN Delivery | Amazon CloudFront |
| State Management | Redux (Auth) + Local State |
| API Communication | Axios + Interceptors |
| Map Integration | AWS Map |
| Reverse Proxy | Nginx |
| Backend Framework | Spring Boot 3.5.6 |
| Programming Language | Java 17 |
| API Layer | REST Controllers |
| Business Layer | Service Interfaces |
| Persistence Layer | JPA + Hibernate |
| Database | Amazon RDS for SQL Server |
| Authentication | JWT + OAuth2 |
| Payment Gateway | VNPay |
| Email Notification | SMTP + Thymeleaf |
| Background Jobs | Scheduler + Async |

---

#### Frontend Architecture

The frontend is structured using a **role-based and domain-oriented approach**:

- **Pages layer**: divided by roles (driver, staff, admin)
- **Components layer**: reusable UI + business components
- **API layer**: domain-based API modules with shared Axios instance
- **Layouts**: separate shells for public users and internal users
- **Routing**: protected routes with role-based access control

Current architecture follows a **layered page-based approach**, with business logic still partially embedded in pages.

Future direction:

- Transition to **feature-based modules**
- Introduce **React Query for server state**
- Standardize API layer and response handling
- Apply lazy loading and code splitting

---

#### Backend Architecture

The backend follows a standard layered architecture:

- Controller → Service → Repository
- Domain modules:
  - Authentication
  - Booking
  - Charging Session
  - Payment
  - Loyalty
  - Compliance
  - Notification

---

### 4. Technical Implementation

#### Workflow Engine Approach

The system uses a **rule-based workflow engine** instead of AI:

- Slot must be available before booking
- Max consecutive slot rules enforced
- Booking confirmation generates QR
- Charging session tracked and billed automatically
- Payment validated via VNPay
- Violations grouped and escalated

Frontend complements this by:

- Managing booking flow and validation UI
- Simulating charging progress (SOC) during session
- Handling multi-step workflows (booking → charging → payment)

---

#### Technical Requirements

- **Frontend**: React + Vite, Axios, JWT, role-based routing
- **Backend**: Spring Boot, REST API, security
- **Database**: Amazon RDS SQL Server
- **Cloud**: AWS S3, CloudFront, EC2, Map
- **Payment**: VNPay
- **Notification**: SMTP
- **State Management**: Redux (auth) + future React Query

---

### 5. Timeline & Milestones

- **Week 7-8**: Core backend + frontend structure setup
- **Week 8**: Booking + session workflows
- **Week 9**: Payment + notification + map integration
- **Week 10**: Loyalty + compliance + admin features
- **Week 11**: Testing + optimization
- **Week 12**: Deployment + go-live

---

### 6. Budget Estimation

The infrastructure cost is estimated based on the proposed AWS architecture, including **Amazon S3, CloudFront, EC2, RDS (SQL Server), Route 53, Amazon SES, and AWS Map (Location Service)**.

The pricing follows a **pay-as-you-go model**, allowing the system to scale with usage. The following estimation reflects a **Minimum Viable Product (MVP)** deployment.

#### Assumptions

- Single AWS region
- One EC2 instance running continuously (730 hours/month)
- One RDS SQL Server instance (Single-AZ)
- Moderate traffic usage
- ~10,000 emails/month
- AWS Map usage within free tier
- No auto-scaling or load balancer included

#### Estimated Monthly Cost

| Service | Configuration | Cost (USD/month) |
|--------|--------------|------------------|
| EC2 | t3.small | ~15.18 |
| RDS SQL Server | db.t3.small | ~26.28 |
| CloudFront | Free tier | 0.00 |
| Route 53 | Hosted zone | ~0.50 |
| Amazon SES | 10,000 emails | ~1.00 |
| AWS Map | Free tier | 0.00 |
| **Total (MVP)** |  | **~42.96 USD/month** |

#### Optional Upgrade

| Service | Upgrade | Additional Cost |
|--------|--------|----------------|
| CloudFront | Pro plan | +15.00 USD |

**Total with upgrade:** ~57.96 USD/month

#### Variable Costs

Additional costs may vary depending on usage:

- Amazon S3 storage
- RDS storage and backup
- Data transfer (CloudFront / EC2)
- Domain registration
- VNPay transaction fees

#### Summary

The system can operate at a low initial cost of approximately:

- **~43 USD/month (MVP)**
- **~58 USD/month (enhanced configuration)**

This cost-efficient setup supports early deployment while allowing scalability for future growth.

---

### 7. Risk Assessment

#### Additional Frontend Risks

- Over-reliance on localStorage/sessionStorage
- Inconsistent API usage across modules
- Large bundle size and performance issues
- Polling overload on backend
- Navigation inconsistency

#### Mitigation

- Introduce React Query
- Standardize API layer
- Implement lazy loading
- Reduce polling frequency
- Improve state management strategy

---

### 8. Expected Outcomes

#### Technical Improvements

- Full-stack integration between frontend and backend
- Improved system performance and UX
- Reduced coupling and better modularization
- More reliable and scalable architecture

#### Long-term Value

- Ready for microservices evolution
- Support mobile apps and external integrations
- Enable analytics and optimization
- Strong foundation for EV ecosystem expansion