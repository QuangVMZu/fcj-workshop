---

title: "Proposal"
date: 2026-03-24
weight: 2
chapter: false
pre: " <b> 2. </b> "
--------------------

# EV Charging Station Management System

# Nền tảng Full-Stack cho vận hành trạm sạc và thanh toán số

### 1. Tóm tắt điều hành (Executive Summary)

EV Charging Station Management System là một nền tảng full-stack được thiết kế nhằm số hóa, chuẩn hóa và tối ưu toàn bộ quy trình vận hành dịch vụ sạc xe điện trong bối cảnh nhu cầu sử dụng EV đang tăng nhanh.

Hệ thống cung cấp một giải pháp tích hợp end-to-end, cho phép người dùng tìm kiếm trạm sạc, đặt lịch theo khung giờ, theo dõi trạng thái phiên sạc theo thời gian thực, thực hiện thanh toán và nhận thông báo một cách liền mạch. Đồng thời, hệ thống hỗ trợ đội ngũ vận hành quản lý tài nguyên trạm sạc, kiểm soát vi phạm, theo dõi hiệu suất và đảm bảo tuân thủ quy định.

Về mặt kỹ thuật, hệ thống được xây dựng theo kiến trúc full-stack hiện đại, trong đó frontend đóng vai trò giao diện tương tác theo từng vai trò (driver, staff, admin), còn backend chịu trách nhiệm xử lý toàn bộ logic nghiệp vụ, đảm bảo tính nhất quán và toàn vẹn dữ liệu.

Hệ thống được triển khai trên nền tảng AWS với các thành phần chính:

* Amazon S3 + CloudFront (WAF) cho phân phối frontend nhanh và an toàn
* Application Load Balancer + EC2 Auto Scaling cho backend có khả năng mở rộng và chịu tải cao
* Amazon RDS SQL Server (Single-AZ) cho lưu trữ dữ liệu
* NAT Gateway cho outbound Internet access từ private subnet
* Amazon SES cho gửi email thông báo
* AWS Location Service cho bản đồ và định vị
* AWS Systems Manager (SSM) + IAM cho truy cập an toàn không cần public IP

Giải pháp này giúp các đơn vị vận hành:

* Tối ưu hóa việc sử dụng trạm sạc
* Giảm chi phí vận hành thông qua tự động hóa
* Nâng cao trải nghiệm người dùng
* Chuẩn hóa hệ thống và giảm technical debt
* Sẵn sàng mở rộng khi quy mô hệ thống tăng trưởng

---

### 2. Vấn đề (Problem Statement)

#### Vấn đề hiện tại

* Không hiển thị slot realtime
* Booking & charging chưa tối ưu
* Chậm invoice & payment
* Khó kiểm soát vi phạm
* Frontend/backend thiếu đồng bộ

#### Vấn đề frontend

* Logic phân tán
* API không chuẩn
* Performance kém

---

#### Giải pháp

* Backend xử lý toàn bộ business logic
* Frontend role-based + domain-based
* API chuẩn hóa JWT
* Modular hóa theo domain

#### Chức năng

* Auth (JWT + Google OAuth)
* Booking / Charging / Payment
* Email (SES)
* Map (AWS Location)

---

#### Lợi ích

* **Tăng hiệu suất**
* **UX tốt hơn**
* **Dễ mở rộng**
* **Stable hơn**

---

### 3. Kiến trúc hệ thống (Solution Architecture)

* Edge: Route 53, CloudFront, WAF
* App: ALB + EC2 ASG (multi-AZ)
* Data: RDS SQL Server (private, single-AZ)
* Support: IAM, SSM, Secrets Manager, CloudWatch

#### Luồng chính:

1. User → Route 53
2. CloudFront → S3
3. Frontend → ALB
4. ALB → EC2
5. EC2 → RDS
6. EC2 → SES / NAT

#### Đặc điểm:

* EC2 private (không public IP)
* Outbound qua NAT Gateway
* Secure access qua SSM
* Secrets quản lý bằng Secrets Manager

![EV Charging System Architecture](/images/2-Proposal/AWS-architecture.png)

---

### 4. Architecture Justification

#### Lý do chọn kiến trúc

**1. CloudFront + S3**

* Giảm latency
* Tăng tốc frontend
* Giảm tải backend

**2. ALB + ASG**

* High Availability
* Auto scaling
* Stateless backend

**3. EC2**

* Phù hợp Spring Boot
* Dễ kiểm soát

**4. RDS Single-AZ**

* Tiết kiệm chi phí
* Upgrade sau

**5. NAT Gateway**

* Outbound secure
* Không cần public IP

**6. SSM + IAM**

* Không cần SSH
* Secure access

**7. Secrets Manager**

* Bảo mật credentials

**8. CloudWatch + CloudTrail**

* Monitoring + audit

---

#### Trade-offs

* Không Multi-AZ DB
* EC2 cần manage
* NAT cost cao

---

### 5. Triển khai kỹ thuật

* React + Axios + JWT
* Spring Boot
* RDS SQL Server
* AWS Cloud
* VNPay
* SES

---

### 6. Timeline

* Week 7–8: Thiết lập hệ thống và kiến trúc
* Week 8: Booking & Charging session
* Week 9: Thanh toán & Tích hợp bản đồ
* Week 10: Loyalty & Compliance
* Week 11: Kiểm thử & Tối ưu hóa
* Week 12: Deployment

---

### 7. Ước tính chi phí (Budget Estimation)

#### Bao gồm:

* S3 + CloudFront
* EC2 (ASG)
* RDS SQL Server
* Route 53
* SES
* NAT Gateway

---

#### Giả định

* 1 region
* 2 EC2
* 1 RDS
* 1 NAT Gateway

---

#### Chi phí

| Service     | Cost        |
| ----------- | ----------- |
| EC2         | ~30         |
| RDS         | ~26         |
| NAT Gateway | ~32–35      |
| Route 53    | ~0.5        |
| SES         | ~1          |
| **Total**   | **~90 USD** |

---

### 8. Risk Assessment

* API chưa chuẩn
* Performance frontend

---

### 9. Expected Outcomes

* Scalable
* Stable
* Production-ready