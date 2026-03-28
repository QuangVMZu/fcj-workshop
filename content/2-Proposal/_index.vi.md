---
title: "Proposal"
date: 2026-03-24
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# EV Charging Station Management System

# Nền tảng Full-Stack cho vận hành trạm sạc và thanh toán số

### 1. Tóm tắt điều hành (Executive Summary)

Hệ thống EV Charging Station Management System là một nền tảng full-stack được thiết kế nhằm hỗ trợ toàn bộ vòng đời vận hành của dịch vụ sạc xe điện, bao gồm cả **xử lý nghiệp vụ backend** và **tương tác người dùng frontend**.

Hệ thống quản lý các quy trình cốt lõi như tìm kiếm trạm sạc, đặt lịch theo khung giờ, xác nhận booking, theo dõi phiên sạc, tạo hóa đơn, xử lý thanh toán và quản lý tuân thủ người dùng trong một kiến trúc thống nhất.

Ở phía backend, hệ thống được xây dựng trên Spring Boot và tích hợp với hạ tầng cloud. Ở phía frontend, hệ thống cung cấp ứng dụng web theo vai trò (driver, staff, admin) với các luồng nghiệp vụ rõ ràng.

Frontend giao tiếp với backend thông qua lớp API chuẩn hóa sử dụng Axios và JWT, bao phủ nhiều domain như authentication, station discovery, booking, charging session, payment, notification và reporting.

Hệ thống tích hợp VNPay cho thanh toán, AWS S3 và CloudFront cho hosting frontend, Amazon RDS cho lưu trữ dữ liệu, AWS Map cho bản đồ và SMTP cho gửi email.

Giải pháp giúp các đơn vị vận hành trạm sạc số hóa quy trình, nâng cao hiệu quả, chuẩn hóa hệ thống và mang lại trải nghiệm người dùng xuyên suốt.

---

### 2. Vấn đề (Problem Statement)

#### Vấn đề hiện tại

Khi số lượng xe điện tăng nhanh, các đơn vị vận hành phải đối mặt với nhiều thách thức:

- Thiếu khả năng hiển thị trạng thái slot theo thời gian thực
- Quy trình booking và phiên sạc chưa tối ưu
- Chậm trễ trong tạo hóa đơn và đối soát thanh toán
- Khó kiểm soát vi phạm (overstay, misuse)
- Frontend và backend chưa tích hợp chặt chẽ
- Trải nghiệm người dùng không đồng nhất giữa các vai trò
- Phụ thuộc nhiều vào localStorage/sessionStorage
- API chưa được chuẩn hóa

#### Vấn đề phía frontend

- Logic nghiệp vụ nằm trong page → khó tái sử dụng
- Gọi API không đồng nhất
- Thiếu quản lý server state
- Bundle lớn, polling nhiều → giảm hiệu năng
- Navigation chưa nhất quán

Nếu không có kiến trúc full-stack chuẩn hóa, hệ thống sẽ khó mở rộng, khó bảo trì và dễ phát sinh lỗi.

---

#### Giải pháp

Giải pháp đề xuất là một **nền tảng full-stack thống nhất**:

- Backend trung tâm xử lý toàn bộ nghiệp vụ
- Frontend theo kiến trúc role-based + domain-based
- API layer chuẩn hóa
- Tách module theo domain (auth, booking, charging, payment...)
- Giảm phụ thuộc browser storage
- Tối ưu hiệu năng frontend

Chức năng chính:

- Authentication và phân quyền
- Booking và quản lý slot
- Quản lý phiên sạc
- Tạo hóa đơn và thanh toán VNPay
- Loyalty và compliance
- Hosting frontend qua S3 + CloudFront
- API routing qua Nginx
- Tích hợp bản đồ AWS Map
- Background jobs

---

#### Lợi ích

- **Tăng hiệu suất sử dụng trạm**
- **Cải thiện trải nghiệm người dùng**
- **Giảm technical debt**
- **Dễ mở rộng hệ thống**
- **Dễ bảo trì**
- **Tăng độ ổn định**

---

### 3. Kiến trúc hệ thống (Solution Architecture)

Hệ thống sử dụng kiến trúc **full-stack nhiều lớp**, tách biệt rõ:

- Frontend
- Backend
- Database

Người dùng truy cập frontend qua CloudFront (S3). Frontend gọi API qua Axios (JWT). Nginx định tuyến request vào backend Spring Boot. Dữ liệu lưu trên Amazon RDS SQL Server.

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

#### Kiến trúc Frontend

- Pages theo role
- Components tái sử dụng
- API layer chuẩn hóa
- Layouts theo role
- Routing có phân quyền

Hướng cải tiến:

- Feature-based architecture
- React Query
- Chuẩn hóa API
- Lazy loading

---

#### Kiến trúc Backend

- Controller → Service → Repository

Modules:

- Authentication
- Booking
- Charging Session
- Payment
- Loyalty
- Compliance
- Notification

---

### 4. Triển khai kỹ thuật (Technical Implementation)

#### Workflow Engine

Hệ thống sử dụng rule-based:

- Slot phải available
- Giới hạn slot liên tiếp
- Booking → QR
- Charging → invoice
- Payment → VNPay
- Violation → xử lý

Frontend:

- Validate flow
- Simulate SOC
- Multi-step workflow

---

#### Yêu cầu kỹ thuật

- Frontend: React + Axios + JWT
- Backend: Spring Boot
- Database: RDS SQL Server
- Cloud: AWS
- Payment: VNPay
- Notification: SMTP

---

### 5. Timeline & Milestones

- Week 7-8: Setup hệ thống
- Week 8: Booking + session
- Week 9: Payment + map
- Week 10: Loyalty + compliance
- Week 11: Testing
- Week 12: Deploy

---

### 6. Ước tính chi phí (Budget Estimation)

Chi phí hạ tầng được ước tính dựa trên kiến trúc AWS đã đề xuất, bao gồm:

- Amazon S3 (lưu trữ frontend)
- CloudFront (CDN)
- EC2 (backend)
- RDS SQL Server (database)
- Route 53 (DNS)
- Amazon SES (email)
- AWS Map (bản đồ)

Hệ thống sử dụng mô hình **pay-as-you-go**, chi phí phụ thuộc vào mức sử dụng thực tế.

---

#### Giả định

- 1 region
- 1 EC2 chạy 24/7 (730 giờ/tháng)
- 1 RDS instance (Single-AZ)
- ~10,000 email/tháng
- Traffic ở mức MVP
- Chưa có autoscaling / load balancer

---

#### Chi phí ước tính hàng tháng

| Service | Cấu hình | Chi phí (USD/tháng) |
|--------|---------|--------------------|
| EC2 | t3.small | ~15.18 |
| RDS SQL Server | db.t3.small | ~26.28 |
| CloudFront | Free tier | 0.00 |
| Route 53 | Hosted zone | ~0.50 |
| Amazon SES | 10,000 email | ~1.00 |
| AWS Map | Free tier | 0.00 |
| **Tổng (MVP)** |  | **~42.96 USD/tháng** |

---

#### Nâng cấp (tuỳ chọn)

| Service | Nâng cấp | Chi phí |
|--------|--------|--------|
| CloudFront | Pro plan | +15 USD |

**Tổng nâng cấp:** ~57.96 USD/tháng

---

#### Chi phí biến đổi

- S3 storage
- RDS storage & backup
- Data transfer
- Domain
- Phí VNPay

---

#### Tổng kết

- **~43 USD/tháng (MVP)**
- **~58 USD/tháng (Production nhẹ)**

Đây là mức chi phí thấp, phù hợp cho giai đoạn triển khai ban đầu và có thể mở rộng trong tương lai.

---

### 7. Risk Assessment

#### Rủi ro frontend

- Phụ thuộc localStorage
- API không đồng nhất
- Bundle lớn
- Polling nhiều
- Navigation lỗi

#### Giảm thiểu

- React Query
- Chuẩn hóa API
- Lazy loading
- Giảm polling
- Tối ưu state

---

### 8. Expected Outcomes

#### Cải thiện kỹ thuật

- Full-stack integration
- UX tốt hơn
- Code sạch hơn
- Dễ mở rộng

#### Giá trị dài hạn

- Sẵn sàng microservices
- Hỗ trợ mobile
- Phân tích dữ liệu
- Mở rộng hệ sinh thái EV