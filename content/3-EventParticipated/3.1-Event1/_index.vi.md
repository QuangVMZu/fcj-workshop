---
title: "Event 1 - Building Full-Stack Observability on AWS with Datadog"
date: 2026-02-27
weight: 1
chapter: false
---

## Tổng quan

Đây là workshop thực hành (hands-on) được tổ chức bởi Datadog phối hợp cùng AWS, tập trung vào việc triển khai **full-stack observability** trong các hệ thống cloud-native hiện đại. 

Khác với các buổi học lý thuyết thông thường, workshop này nhấn mạnh vào việc **thực hành trực tiếp trong môi trường sandbox**, mô phỏng các tình huống vận hành thực tế của hệ thống production.

Mục tiêu chính là giúp người tham gia hiểu cách **quan sát, debug và tối ưu hệ thống phân tán** thông qua việc hợp nhất metrics, logs và traces vào một nền tảng duy nhất.

---

## Kiến thức trọng tâm

### 1. Từ Monitoring đến Observability

Monitoring truyền thống trả lời:
> “Hệ thống có đang hoạt động không?”

Observability trả lời:
> “Tại sao hệ thống lại hoạt động như vậy?”

Ba thành phần cốt lõi:

- **Metrics** → Hiệu năng hệ thống (CPU, RAM, latency)
- **Logs** → Nhật ký chi tiết của hệ thống
- **Traces** → Luồng request xuyên suốt nhiều service

👉 Insight quan trọng:  
> Observability là việc **kết nối cả 3 lớp dữ liệu** để hiểu rõ hành vi hệ thống.

---

### 2. Kiến trúc Observability hợp nhất

Trong workshop, hệ thống được xây dựng theo mô hình:

- AWS services tạo telemetry data
- Datadog đóng vai trò nền tảng quan sát trung tâm
- Dữ liệu được hiển thị qua dashboard và alert

**Luồng dữ liệu:**
Application → Logs / Metrics / Traces → Datadog → Dashboard / Alert


Mô hình này giúp:

- Phát hiện bất thường nhanh chóng
- Theo dõi lỗi xuyên suốt nhiều service
- Giảm thời gian xử lý sự cố (MTTR)

---

### 3. Distributed Tracing & Root Cause Analysis

Một phần quan trọng của workshop là tracing request qua nhiều service.

Thực hành gồm:

- Theo dõi request từ frontend → backend → database
- Xác định bottleneck về hiệu năng
- Tìm chính xác điểm lỗi

💡 Insight:
> Trong hệ thống phân tán, **lỗi thường không nằm ở nơi nó xuất hiện**.

---

### 4. Debug hệ thống theo thời gian thực

Trong môi trường sandbox, chúng tôi:

- Giả lập lỗi hiệu năng
- Trigger alert
- Phân tích logs và traces

Qua đó hiểu được:

- Cách cấu hình alert hiệu quả (tránh noise)
- Debug mà không cần truy cập trực tiếp server
- Quan sát thay thế cho việc “đoán lỗi”

---

## Bài học thực tiễn

- Thiết kế hệ thống với **observability ngay từ đầu**
- Sử dụng **metrics + logs + traces kết hợp**, không tách rời
- Đầu tư vào **dashboard và alerting**
- Observability đặc biệt quan trọng với:
  - Microservices
  - Serverless
  - Hệ thống quy mô lớn

---

## Cảm nhận cá nhân

Workshop giúp mình chuyển tư duy từ:

> “Làm sao để monitor hệ thống?”

sang:

> “Làm sao để hiểu hệ thống trong mọi tình huống?”

Ngoài ra, mình nhận ra rằng:

- Developer hiện đại không chỉ viết code
- Mà còn chịu trách nhiệm về:
  - Reliability
  - Observability
  - Operability

---

## Ứng dụng thực tế

Sau workshop, mình bắt đầu:

- Thiết kế logging strategy ngay từ backend
- Suy nghĩ về tracing khi thiết kế hệ thống
- Đưa monitoring vào từ giai đoạn thiết kế

Điều này ảnh hưởng trực tiếp đến cách mình xây dựng các ứng dụng trên AWS.  