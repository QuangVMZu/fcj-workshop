---
title: "Worklog Tuần 9"
date: 2026-03-09
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu Tuần 9

Tuần này tập trung vào việc triển khai thử nghiệm (deploy test) dự án, kiểm tra toàn bộ hệ thống và xử lý lỗi phát sinh trong quá trình phát triển và kiểm thử.

- **Triển khai hệ thống**:
  - Deploy ứng dụng lên môi trường thực tế (EC2)
  - Cấu hình các service liên quan (RDS, S3, Map)
  - Kiểm tra khả năng kết nối giữa các thành phần

- **Kiểm thử hệ thống**:
  - Test luồng chức năng chính của ứng dụng
  - Kiểm tra giao diện và trải nghiệm người dùng
  - Xác minh dữ liệu giữa frontend và backend

- **Sửa lỗi và tối ưu**:
  - Fix bug phát sinh trong quá trình test
  - Tối ưu code và cấu trúc hệ thống
  - Cải thiện hiệu năng và độ ổn định

- **Hoàn thiện hệ thống**:
  - Đảm bảo hệ thống hoạt động ổn định
  - Chuẩn bị cho giai đoạn demo hoặc release

---

### Tổng quan công việc

| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:----:|---------|:------------:|:---------------:|-------------------|
|  2   | **Deployment Setup**:<br>- Deploy ứng dụng lên EC2<br>- Cấu hình môi trường | 09/03/2026 | 09/03/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  3   | **System Testing**:<br>- Test chức năng chính<br>- Kiểm tra luồng dữ liệu | 10/03/2026 | 10/03/2026 | Internal |
|  4   | **Bug Fixing**:<br>- Phát hiện và sửa lỗi<br>- Debug hệ thống | 11/03/2026 | 11/03/2026 | Internal |
|  5   | **Optimization**:<br>- Tối ưu code<br>- Cải thiện hiệu năng | 12/03/2026 | 12/03/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  6   | **Final Testing**:<br>- Kiểm tra lại toàn bộ hệ thống<br>- Chuẩn bị demo | 13/03/2026 | 13/03/2026 | Internal |

---

### Kết quả đạt được

#### Những gì đã hoàn thành

- Deploy thành công hệ thống lên **Amazon EC2**
- Kết nối ổn định với các dịch vụ:
  - **Amazon RDS**
  - **Amazon S3**
  - **AWS Map**
- Kiểm thử được toàn bộ luồng chức năng chính của hệ thống
- Phát hiện và xử lý các lỗi trong quá trình test
- Tối ưu hiệu năng và cải thiện trải nghiệm người dùng
- Hoàn thiện hệ thống ở mức **sẵn sàng demo**

#### Tóm tắt kiến trúc

- **Compute**: EC2 (Deploy ứng dụng)
- **Database**: RDS (SQL Server)
- **Storage**: S3 (Static files)
- **Map**: AWS Map (Location service)
- **Flow**: Frontend → Backend → AWS Services
- **Stage**: Testing / Pre-production
