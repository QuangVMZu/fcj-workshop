---
title: "Worklog Tuần 8"
date: 2026-03-02
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu Tuần 8

Tuần này tập trung vào việc bắt đầu triển khai dự án thực tế bằng cách phát triển mã nguồn và tích hợp các dịch vụ AWS cốt lõi vào hệ thống.

- **Phát triển dự án**:
  - Bắt đầu code các chức năng chính của dự án
  - Kết nối giữa frontend, backend và database
  - Hoàn thiện nền tảng kỹ thuật ban đầu cho hệ thống

- **Tích hợp dịch vụ AWS**:
  - **Amazon EC2** để triển khai và vận hành ứng dụng
  - **Amazon RDS** để lưu trữ và quản lý dữ liệu quan hệ
  - **Amazon S3** để lưu trữ file tĩnh và tài nguyên người dùng
  - **AWS Map** để hỗ trợ hiển thị bản đồ và dữ liệu vị trí

- **Kiểm thử và hiệu chỉnh**:
  - Kiểm tra luồng hoạt động giữa các thành phần
  - Xử lý các lỗi phát sinh trong quá trình tích hợp
  - Cải thiện cấu trúc dự án để thuận tiện cho các tuần tiếp theo

---

### Tổng quan công việc

| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:----:|---------|:------------:|:---------------:|-------------------|
|  2   | **Project Coding Setup**:<br>- Khởi tạo cấu trúc project<br>- Bắt đầu code các chức năng chính | 02/03/2026 | 02/03/2026 | Internal |
|  3   | **Database & Backend Integration**:<br>- Kết nối backend với RDS<br>- Xử lý dữ liệu cơ bản | 03/03/2026 | 03/03/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  4   | **S3 Integration**:<br>- Cấu hình S3<br>- Upload và quản lý file tĩnh | 04/03/2026 | 04/03/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  5   | **Map Service Integration**:<br>- Tích hợp bản đồ AWS<br>- Hiển thị dữ liệu vị trí trên giao diện | 05/03/2026 | 05/03/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  6   | **EC2 Deployment & Testing**:<br>- Deploy ứng dụng lên EC2<br>- Kiểm thử tích hợp toàn hệ thống | 06/03/2026 | 06/03/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |

---

### Kết quả đạt được

#### Những gì đã hoàn thành

- Bắt đầu triển khai **mã nguồn thực tế** cho dự án
- Xây dựng được nền tảng tích hợp giữa **frontend, backend và database**
- Kết nối thành công **Amazon RDS** để lưu trữ dữ liệu
- Cấu hình và sử dụng **Amazon S3** để lưu trữ tài nguyên tĩnh
- Tích hợp **AWS Map** để phục vụ chức năng hiển thị vị trí
- Triển khai ứng dụng thử nghiệm trên **Amazon EC2**
- Kiểm tra được luồng hoạt động cơ bản của hệ thống sau tích hợp

#### Tóm tắt kiến trúc

- **Compute**: Amazon EC2 để chạy ứng dụng
- **Database**: Amazon RDS để lưu trữ dữ liệu quan hệ
- **Storage**: Amazon S3 để lưu file và tài nguyên tĩnh
- **Map Service**: AWS Map để hiển thị bản đồ và dữ liệu vị trí
- **Application Flow**: Frontend ↔ Backend ↔ AWS Services
