---
title: "Event 2 - AWS re:Invent Recap HCMC"
date: 2026-02-04
weight: 2
chapter: false
---

# AWS re:Invent Recap HCMC

## Tổng quan

AWS re:Invent Recap HCMC là sự kiện cộng đồng tổng hợp những thông tin quan trọng từ AWS re:Invent — hội nghị cloud lớn nhất của AWS trên toàn cầu.

Sự kiện mang đến cái nhìn tổng quan về:
- Các dịch vụ AWS mới
- Xu hướng kiến trúc cloud hiện đại
- Best practices từ thực tế triển khai hệ thống

---

## Chủ đề và kiến thức nổi bật

### 1. Cloud đang tiến tới mức độ trừu tượng cao hơn

Một xu hướng rõ ràng:

> Developer tập trung vào logic, không còn phải quản lý hạ tầng

Ví dụ:

- Serverless (Lambda, API Gateway)
- Managed services (RDS, DynamoDB)
- AI/ML (Bedrock, SageMaker)

💡 Insight:
> Tương lai của cloud là **tự động hóa nhiều hơn, quản lý ít hơn**

---

### 2. Các mô hình kiến trúc hiện đại

Các pattern phổ biến:

- **Microservices**
- **Event-driven architecture**
- **Serverless-first**

Luồng tiêu biểu:
Client → API Gateway → Lambda → Database (RDS/DynamoDB)


Ưu điểm:

- Dễ scale
- Tách biệt lỗi
- Tăng tốc phát triển

---

### 3. Tối ưu chi phí là trách nhiệm của kỹ sư

Không chỉ là bài toán tài chính, mà là bài toán thiết kế.

Chiến lược:

- Right-sizing tài nguyên
- Savings Plans / Reserved Instances
- Chọn đúng service

💡 Insight:
> Hệ thống tốt là hệ thống **tối ưu cả hiệu năng và chi phí**

---

### 4. Độ tin cậy và khả năng chịu lỗi

AWS nhấn mạnh:

- Multi-AZ deployment
- Backup
- Fault tolerance

Nguyên tắc:
> “Mọi thứ đều có thể fail — hãy thiết kế cho điều đó.”

---

### 5. AI trở thành một phần mặc định của Cloud

AI không còn là optional.

Các điểm nổi bật:

- Amazon Bedrock
- AI trong quy trình phát triển
- Agent automation

---

## Bài học thực tiễn

- Ưu tiên **managed services**
- Thiết kế hệ thống theo:
  - Scalability
  - Reliability
  - Cost efficiency
- Tư duy theo **architecture patterns**
- Kết hợp nhiều service thay vì over-engineering

---

## Cảm nhận cá nhân

Sự kiện giúp mình chuyển từ:

> “Học từng service riêng lẻ”

sang:

> “Tư duy theo hệ thống và kiến trúc”

Ngoài ra mình nhận ra:

- Hệ thống thực tế luôn là sự kết hợp nhiều service
- Kiến trúc tốt là bài toán **trade-off**, không phải hoàn hảo

---

## Ứng dụng thực tế

Sau sự kiện, mình bắt đầu:

- Thiết kế hệ thống theo pattern
- Tính toán cost ngay từ đầu
- Kết hợp các service như:
  - EC2 + RDS + S3
  - hoặc serverless stack

Tư duy này ảnh hưởng trực tiếp đến cách mình triển khai dự án thực tế.