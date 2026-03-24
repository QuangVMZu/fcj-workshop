---
title: "Worklog Tuần 2"
date: 2026-01-12
weight: 2 
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu Tuần 2

Tuần này tập trung vào việc xây dựng nền tảng về quản lý truy cập và kiến trúc mạng trong AWS, từ cơ bản đến nâng cao. Các mục tiêu chính bao gồm:

- **Quản lý IAM**:
  - Tạo Users và Groups
  - Gắn Policies phù hợp
  - Tạo và sử dụng IAM Roles
  - Thực hành **Switch Role**

- **Nắm vững VPC Networking**:
  - Hiểu tổng quan kiến trúc VPC
  - So sánh **Security Groups vs Network ACLs**
  - Chuẩn bị môi trường mạng cho EC2

- **Triển khai EC2**:
  - Launch EC2 trong subnet
  - Cấu hình Security Group
  - Kết nối SSH bằng Key Pair

- **Cấu hình Hybrid DNS với Route 53 Resolver**:
  - Tạo Key Pair
  - Cấu hình Security Group
  - Cấu hình DNS:
    - Tạo Outbound Endpoint
    - Tạo Resolver Rules
    - Tạo Inbound Endpoint
  - Dọn dẹp tài nguyên

- **Thiết lập VPC Peering**:
  - Giới thiệu cơ chế Peering
  - Sử dụng CloudFormation để khởi tạo hạ tầng
  - Tạo Security Group & EC2
  - Cập nhật Network ACLs
  - Thiết lập Peering Connection
  - Cập nhật Route Tables
  - Bật Cross-Peer DNS

- **Triển khai AWS Transit Gateway**:
  - Tạo Transit Gateway
  - Tạo Attachments cho các VPC
  - Cấu hình Route Tables cho Transit Gateway
  - Cập nhật Route Tables của VPC

---

### Tổng quan công việc

| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:----:|---------|:------------:|:---------------:|-------------------|
|  2   | **IAM & EC2 cơ bản**:<br>- Tạo Users, Groups<br>- Gắn Policies<br>- Tạo Role & Switch Role<br>- Launch EC2 + SSH | 12/01/2026 | 12/01/2026 | [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj/) |
|  3   | **VPC Networking**:<br>- Tổng quan VPC<br>- So sánh Security Group vs NACL<br>- Chuẩn bị subnet cho EC2 | 13/01/2026 | 13/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  4   | **Hybrid DNS (Route 53 Resolver)**:<br>- Tạo Key Pair<br>- Cấu hình Security Group<br>- Tạo Outbound / Inbound Endpoint<br>- Tạo Resolver Rules | 14/01/2026 | 14/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  5   | **VPC Peering**:<br>- Tạo CloudFormation Template<br>- Tạo EC2 & SG<br>- Thiết lập Peering<br>- Update Route Tables<br>- Enable DNS | 15/01/2026 | 15/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  6   | **Transit Gateway**:<br>- Tạo TGW<br>- Attach VPC<br>- Cấu hình Route Tables | 16/01/2026 | 16/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |

---

### Kết quả đạt được

#### Những gì đã hoàn thành

- Hiểu rõ cơ chế phân quyền với **IAM** và thực hành thành công **Switch Role**
- Nắm vững kiến trúc mạng **VPC** và cơ chế bảo mật với Security Group
- Triển khai thành công **EC2** và kết nối SSH
- Cấu hình thành công **Hybrid DNS** với Route 53 Resolver
- Triển khai **VPC Peering** để kết nối Dev và Staging
- Triển khai **AWS Transit Gateway** làm trung tâm kết nối nhiều VPC
- Làm quen với **Infrastructure as Code** thông qua CloudFormation
- Thực hiện **dọn dẹp tài nguyên** sau khi hoàn thành

#### Tóm tắt kiến trúc

- **Phân quyền IAM**: Users → Groups → Policies → Roles
- **Mạng VPC**:
  - Security Group: kiểm soát instance
  - NACL: kiểm soát subnet
- **Hybrid DNS**:
  - On-prem ↔ AWS qua Route 53 Resolver
- **Kết nối mạng**:
  - VPC Peering: point-to-point
  - Transit Gateway: hub-and-spoke
