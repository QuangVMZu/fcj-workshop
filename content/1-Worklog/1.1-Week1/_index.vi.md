---
title: "Worklog Tuần 1"
date: 2026-01-05
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu Tuần 1

Tuần này tập trung vào việc làm quen với AWS, thiết lập môi trường ban đầu và xây dựng nền tảng về quản lý tài khoản và chi phí. Các mục tiêu chính bao gồm:

- **Tìm hiểu tổng quan AWS**:
  - Hiểu cách AWS hoạt động và mô hình Cloud
  - Tìm hiểu các dịch vụ cốt lõi (EC2, S3, IAM, VPC,...)

- **Làm quen môi trường lab**:
  - Hiểu cấu trúc bài lab
  - Làm quen với quy trình thực hành

- **Làm việc nhóm**:
  - Lập nhóm
  - Làm quen và giao lưu với các thành viên

- **Thiết lập tài khoản AWS**:
  - Tạo tài khoản AWS (Free Tier)
  - Bật **MFA cho Root User**
  - Tạo IAM User quản trị
  - Tạo Group và gán quyền Administrator
  - Làm quen AWS Console

- **Quản lý chi phí với AWS Budgets**:
  - Tạo Cost Budget
  - Tạo Usage Budget
  - Tạo Reservation Budget
  - Tạo Savings Plans Budget
  - Cấu hình cảnh báo
  - Dọn dẹp tài nguyên

---

### Tổng quan công việc

| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|:----:|----------|:------------:|:---------------:|-------------------|
|  2   | **Tìm hiểu AWS cơ bản**:<br>- Tổng quan AWS<br>- Mô hình Cloud<br>- Các dịch vụ chính | 05/01/2026 | 05/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  3   | **Làm quen Lab**:<br>- Cấu trúc bài lab<br>- Quy trình thực hành | 06/01/2026 | 06/01/2026 | [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj/) |
|  4   | **Làm việc nhóm**:<br>- Lập nhóm<br>- Làm quen thành viên<br>- Định hướng học tập | 07/01/2026 | 07/01/2026 | Nội bộ |
|  5   | **Thiết lập AWS & IAM**:<br>- Tạo account<br>- Bật MFA<br>- Tạo IAM User + Admin Group<br>- Làm quen Console | 08/01/2026 | 08/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  6   | **AWS Budgets**:<br>- Tạo Cost/Usage/Reservation/Savings Plans Budget<br>- Cấu hình cảnh báo<br>- Dọn dẹp tài nguyên | 09/01/2026 | 09/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |

---

### Kết quả đạt được

#### Những gì đã hoàn thành

- Hiểu rõ tổng quan về **AWS** và các dịch vụ chính
- Nắm được cách triển khai và thực hành các **bài lab**
- Tham gia **làm việc nhóm**, kết nối với các thành viên
- Tạo thành công tài khoản AWS với **Free Tier (~$100 credit)**
- Bật **MFA cho Root User** để tăng bảo mật
- Tạo **IAM User riêng** để tránh sử dụng Root Account
- Làm quen với **AWS Console**
- Thiết lập **AWS Budgets** để kiểm soát chi phí
- Cấu hình **cảnh báo chi phí** và dọn dẹp tài nguyên

#### Tóm tắt nguyên tắc

- **Bảo mật**:
  - Không dùng Root cho daily work
  - Luôn bật MFA

- **Chi phí**:
  - Theo dõi bằng AWS Budgets
  - Alert khi vượt ngưỡng

- **Học tập**:
  - Thực hành lab
  - Làm việc nhóm
