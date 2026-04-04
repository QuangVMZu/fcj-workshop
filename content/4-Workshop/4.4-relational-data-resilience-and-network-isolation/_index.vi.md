---
title: "Độ bền dữ liệu quan hệ và cô lập mạng"
date: 2026-04-04
weight: 4
chapter: false
pre: " <b> 4.4. </b> "
---

# Thực hành độ bền dữ liệu quan hệ và cô lập mạng

## Tổng quan

- Phần này giải thích cách nền tảng lưu dữ liệu giao dịch trong Amazon RDS trong khi vẫn giữ tầng dữ liệu ở trạng thái private.
- Nội dung tập trung vào hành vi primary và standby, kết nối database từ EC2, và các bước kiểm tra vận hành trước khi tác động tới dữ liệu production.
- Đây là phần nên đọc khi ứng dụng không kết nối được database, hành vi failover chưa rõ, hoặc nhóm cần xác minh giả định về backup và network isolation.

## Các dịch vụ AWS trong phần này

### Amazon RDS
![Amazon RDS](/images/4-Workshop/rds-overview.png)
- Dịch vụ cơ sở dữ liệu quan hệ được quản lý cho tầng dữ liệu của hệ thống.
- Tự động xử lý provisioning, backup, patching và failover.

### Primary Database Instance
![RDS Primary Instance](/images/4-Workshop/rds-primary.png)
- Xử lý workload giao dịch chính.
- Thực hiện các thao tác đọc/ghi từ ứng dụng.

### Standby Database Instance
- Duy trì bản sao đồng bộ với instance chính.
- Tự động chuyển sang hoạt động khi xảy ra failover.

### Không có truy cập công khai
- Đảm bảo database không thể truy cập trực tiếp từ internet.
- Chỉ cho phép truy cập từ các resource nội bộ trong VPC.

### Kết nối EC2 tới RDS
- Application server kết nối tới database qua mạng private.
- Thường sử dụng security group và endpoint nội bộ.

## Vì sao thiết kế này dùng Amazon RDS

- Kiến trúc đang mô hình hóa dữ liệu giao dịch dạng quan hệ, nên Amazon RDS là lựa chọn lưu trữ được thể hiện rõ.
- Proposal của dự án nhắc đến Amazon RDS for SQL Server, nhưng sơ đồ chỉ ghi Amazon RDS. Luôn xác nhận engine thật trước khi chạy bước chuyên biệt theo engine.
- Không có DynamoDB, cache tier hay analytics warehouse trong sơ đồ, nên workshop này chỉ tập trung vào đường kết nối quan hệ private từ EC2 tới RDS.

## Luồng AWS từng bước

1. Backend chạy trên EC2 cần đọc hoặc ghi dữ liệu nghiệp vụ.
   Dịch vụ: Amazon EC2.
   Điều xảy ra: Ứng dụng mở kết nối tới RDS endpoint qua mạng VPC.
   Tại sao quan trọng: Database traffic nên chỉ đi trên đường mạng private.
2. Request đến primary RDS instance.
   Dịch vụ: Amazon RDS.
   Điều xảy ra: Primary DB instance xử lý transaction của ứng dụng.
   Tại sao quan trọng: Ghi dữ liệu và phần lớn đọc vận hành nên đi vào primary nếu chưa có read replica.
3. RDS giữ một bản standby được đồng bộ.
   Dịch vụ: Amazon RDS.
   Điều xảy ra: Thay đổi được sao chép sang môi trường standby.
   Tại sao quan trọng: Standby giúp tăng độ bền khi primary hoặc hạ tầng bên dưới gặp lỗi.
4. Database không thể bị truy cập trực tiếp từ internet.
   Dịch vụ: RDS, private subnet, security group.
   Điều xảy ra: Chỉ các đường kết nối được phê duyệt từ tầng ứng dụng mới truy cập được.
   Tại sao quan trọng: Giảm bề mặt tấn công và hạn chế việc quét từ internet.
5. Khi primary gặp lỗi, RDS có thể chuyển sang đường standby.
   Dịch vụ: Amazon RDS.
   Điều xảy ra: Failover diễn ra ở tầng database được quản lý.
   Tại sao quan trọng: Ứng dụng nên kết nối qua RDS endpoint thay vì một hostname cứng theo máy cụ thể.

## AWS CLI Walkthrough

### 1. Kiểm tra DB instance

```bash
aws rds describe-db-instances \
  --db-instance-identifier ${DB_INSTANCE_ID} \
  --region ${AWS_REGION}
```

### 2. Hiển thị trạng thái, cờ Multi-AZ và endpoint

```bash
aws rds describe-db-instances \
  --db-instance-identifier ${DB_INSTANCE_ID} \
  --region ${AWS_REGION} \
  --query 'DBInstances[0].[DBInstanceStatus,MultiAZ,Endpoint.Address,Endpoint.Port]' \
  --output table
```

### 3. Thực hiện failover test ở môi trường không phải production đã được phê duyệt

```bash
aws rds reboot-db-instance \
  --db-instance-identifier ${DB_INSTANCE_ID} \
  --force-failover \
  --region ${AWS_REGION}
```

Chỉ dùng lệnh này khi:

- instance đã bật Multi-AZ,
- chủ môi trường đã phê duyệt bài test,
- nhóm ứng dụng sẵn sàng quan sát hành vi reconnect.

## Ví dụ cấu hình

### Mô hình truy cập database được khuyến nghị

| Hạng mục | Thiết lập khuyến nghị | Ý nghĩa |
| --- | --- | --- |
| DB subnet group | Private subnet ở ít nhất hai Availability Zone | Hỗ trợ isolation và đặt standby |
| Public accessibility | Tắt | Ngăn truy cập trực tiếp từ internet |
| Nguồn truy cập | Chỉ cho phép từ security group của EC2 ứng dụng | Giới hạn chính xác bên được mở session |
| Database port | `${DB_PORT}` theo engine đang dùng | Giúp rule rõ ràng và dễ kiểm toán |
| Điểm kết nối | Dùng DNS endpoint của RDS, không dùng IP của instance | Cho phép failover mà không sửa cấu hình app |

### Ví dụ thông tin kết nối lưu trong Secrets Manager

```json
{
  "host": "ev-prod-rds.xxxxxx.ap-southeast-1.rds.amazonaws.com",
  "port": 1433,
  "database": "evcharging",
  "username": "app_user",
  "password": "change-me"
}
```

## Những gì cần xác minh

- RDS instance không public accessible.
- Ứng dụng kết nối bằng RDS endpoint, không phải private IP cố định.
- DB subnet group dùng private subnet.
- Chỉ security group của ứng dụng được phép tới database port.
- Quy trình failover đã được thử ở ngoài production và có tài liệu ghi lại.

## Lỗi thường gặp

- Mở database cho dải CIDR rộng thay vì chỉ cho security group của ứng dụng.
- Lưu hostname database trực tiếp trong source code thay vì secret được quản lý.
- Hiểu nhầm standby là query endpoint trong khi sơ đồ chỉ cho thấy cặp primary và standby.
- Chạy failover test cưỡng bức mà không thông báo trước cho nhóm ứng dụng.

## Ghi chú / Giả định

- Sơ đồ cho thấy rất rõ mô hình primary và standby kiểu Multi-AZ, nhưng không nêu chính xác engine hay deployment mode của RDS.
- Không có read replica trong hình.
- Phần này không giả định DynamoDB hay Aurora vì chúng không xuất hiện trong sơ đồ kiến trúc.


