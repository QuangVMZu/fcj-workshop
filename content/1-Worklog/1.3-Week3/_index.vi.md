---
title: "Worklog Tuần 3"
date: 2026-01-19
weight: 3 
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu Tuần 3

Tuần này tập trung vào việc phát triển ý tưởng dự án cuối kỳ và thực hành các dịch vụ cốt lõi của AWS như EC2, S3, IAM và RDS. Các mục tiêu chính bao gồm:

- **Phát triển dự án cuối kỳ**:
  - Lên ý tưởng dự án
  - Phân tích và lựa chọn các dịch vụ AWS phù hợp
  - Xây dựng kiến trúc sơ bộ

- **Thực hành EC2**:
  - Tìm hiểu dịch vụ EC2
  - Triển khai máy ảo
  - Cấu hình instance
  - Tạo và sử dụng AMI tùy chỉnh

- **Quản lý truy cập & môi trường phát triển**:
  - Tìm hiểu và thực hành IAM Role
  - Làm quen với Cloud9

- **Lưu trữ và quản lý dữ liệu**:
  - Cấu hình Amazon S3
  - Quản lý dữ liệu và quyền truy cập

- **Cơ sở dữ liệu với RDS**:
  - Tạo database
  - Quản lý và vận hành RDS

- **Ôn tập & củng cố kiến thức**:
  - Ôn lại IAM, EC2, S3, RDS
  - Tổng hợp kiến thức từ Tuần 1 và 2

---

### Tổng quan công việc

| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:----:|---------|:------------:|:---------------:|-------------------|
|  2   | **Định hướng dự án**:<br>- Lên ý tưởng<br>- Chọn dịch vụ AWS<br>- Phác thảo kiến trúc | 19/01/2026 | 19/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  3   | **EC2 & AMI**:<br>- Triển khai EC2<br>- Cấu hình máy ảo<br>- Tạo AMI tùy chỉnh | 20/01/2026 | 20/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  4   | **IAM Role & Cloud9**:<br>- Thực hành IAM Role<br>- Làm quen Cloud9 | 21/01/2026 | 21/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  5   | **S3 Storage**:<br>- Cấu hình S3<br>- Public access<br>- Versioning | 22/01/2026 | 22/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  6   | **RDS & Review**:<br>- Tạo database RDS<br>- Quản lý DB<br>- Ôn tập tổng hợp | 23/01/2026 | 23/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |

---

### Kết quả đạt được

#### Những gì đã hoàn thành

- Xây dựng ý tưởng và kiến trúc cho **dự án cuối kỳ**
- Thành thạo triển khai và quản lý nhiều **EC2 instances**
- Tạo và sử dụng **AMI tùy chỉnh**
- Hiểu và áp dụng **IAM Role** trong thực tế
- Làm quen môi trường phát triển **Cloud9**
- Xây dựng thành công **website tĩnh trên S3**:
  - Public access
  - Versioning
  - Replication (multi-region)
- Tạo và vận hành cơ sở dữ liệu trên **Amazon RDS**
- Củng cố kiến thức từ các tuần trước

#### Tóm tắt kiến trúc

- **Compute**: EC2 + AMI tùy chỉnh
- **Storage**: S3 (Static Website + Versioning + Replication)
- **Database**: RDS (Managed DB)
- **Access Control**: IAM Role
- **Dev Environment**: Cloud9
