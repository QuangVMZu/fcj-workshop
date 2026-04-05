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
- Ở góc nhìn vận hành thực tế, đây là nơi xác định ranh giới persistence của booking, charging session, payment và các nghiệp vụ cốt lõi khác.

## Các dịch vụ AWS trong phần này

### Amazon RDS
![Amazon RDS](/images/4-Workshop/rds-overview.png)
- Dịch vụ cơ sở dữ liệu quan hệ được quản lý cho tầng dữ liệu của hệ thống.
- Tự động xử lý provisioning, backup, patching và failover.
- Giảm đáng kể gánh nặng vận hành so với việc tự quản lý database trên EC2.

### Primary Database Instance
![RDS Primary Instance](/images/4-Workshop/rds-primary.png)
- Xử lý workload giao dịch chính.
- Thực hiện các thao tác đọc/ghi từ ứng dụng.
- Là endpoint chính cho các thay đổi dữ liệu mang tính nghiệp vụ.

### Standby Database Instance
- Duy trì bản sao đồng bộ với instance chính.
- Tự động chuyển sang hoạt động khi xảy ra failover.
- Tồn tại nhằm tăng độ bền chứ không phải để phục vụ traffic trực tiếp từ người dùng.

### Không có truy cập công khai
- Đảm bảo database không thể truy cập trực tiếp từ internet.
- Chỉ cho phép truy cập từ các resource nội bộ trong VPC.
- Giảm bề mặt tấn công và tránh lỗi mở CIDR quá rộng.

### Kết nối EC2 tới RDS
- Application server kết nối tới database qua mạng private.
- Thường sử dụng security group và endpoint nội bộ.
- Giữ cho hợp đồng kết nối giữa app và database ổn định ngay cả khi node vật lý thay đổi trong lúc bảo trì hoặc failover.

## Vì sao thiết kế này dùng Amazon RDS

- Kiến trúc đang mô hình hóa dữ liệu giao dịch dạng quan hệ, nên Amazon RDS là lựa chọn lưu trữ được thể hiện rõ.
- Workshop đã mô tả data layer theo mô hình primary và standby, phù hợp với một tư thế HA dựa trên managed service.
- Không có DynamoDB, cache tier hay analytics database trong sơ đồ hiện tại, nên phần này chỉ tập trung vào đường quan hệ private từ EC2 tới RDS.
- RDS giúp đội ngũ dành ít thời gian hơn cho các tác vụ database mang tính hạ tầng như backup, patching và failover.

## Luồng AWS từng bước

1. Backend chạy trên EC2 cần đọc hoặc ghi dữ liệu nghiệp vụ.
   Dịch vụ: Amazon EC2.
   Điều xảy ra: Ứng dụng mở kết nối tới managed RDS endpoint và thực hiện giao dịch.
   Tại sao quan trọng: Ứng dụng nên phụ thuộc vào endpoint được quản lý, không phải vào một host cụ thể.
2. Request đến primary RDS instance.
   Dịch vụ: Amazon RDS.
   Điều xảy ra: Primary DB instance xử lý transaction của ứng dụng.
   Tại sao quan trọng: Đây là điểm ghi dữ liệu cốt lõi của hệ thống.
3. RDS giữ một bản standby được đồng bộ.
   Dịch vụ: Amazon RDS.
   Điều xảy ra: Trạng thái dữ liệu được sao chép theo availability mode đã chọn.
   Tại sao quan trọng: Đây là nền tảng cho failover và độ bền của dữ liệu.
4. Database không thể bị truy cập trực tiếp từ internet.
   Dịch vụ: RDS, private subnet, security group.
   Điều xảy ra: Chỉ các đường kết nối nội bộ đã được phê duyệt mới tới được database port.
   Tại sao quan trọng: Data tier vẫn được bảo vệ kể cả khi public endpoint khác của hệ thống gặp sự cố.
5. Khi primary gặp lỗi, RDS có thể chuyển sang đường standby.
   Dịch vụ: Amazon RDS.
   Điều xảy ra: RDS chuyển vai trò primary phía sau managed endpoint.
   Tại sao quan trọng: Ứng dụng cần reconnect được mà không phải đổi cấu hình thủ công.

## Hành vi request và response

- Backend gửi SQL request như một phần của workflow nghiệp vụ đồng bộ.
- Primary xử lý đọc/ghi bình thường, còn standby giữ vai trò tăng độ bền thay vì phục vụ traffic trực tiếp.
- Ứng dụng nhận kết quả query, xác nhận transaction hoặc database error và phải xử lý chúng đúng cách.
- Trong lúc bảo trì hoặc failover, timeout, connection pool và retry logic trở thành một phần của hợp đồng vận hành của ứng dụng.

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

Chỉ dùng lệnh failover khi:

- chế độ Multi-AZ hoặc tương đương thực sự đã bật,
- chủ môi trường đã phê duyệt bài test,
- nhóm ứng dụng sẵn sàng quan sát hành vi reconnect.

## Ví dụ cấu hình

### Mô hình truy cập database được khuyến nghị

| Hạng mục | Thiết lập khuyến nghị | Ý nghĩa |
| --- | --- | --- |
| DB subnet group | Private subnet ở ít nhất hai Availability Zone | Hỗ trợ isolation và placement của standby |
| Public accessibility | Tắt | Ngăn truy cập trực tiếp từ internet |
| Nguồn truy cập | Chỉ cho phép từ security group của EC2 ứng dụng | Giới hạn chính xác bên được mở session |
| Database port | `${DB_PORT}` theo engine đang dùng | Làm rule rõ ràng và dễ kiểm toán |
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

## Key Logic / Rules

- Primary database là endpoint chính cho giao dịch bình thường của ứng dụng.
- Standby database giúp tăng độ bền nhưng không phải user-facing endpoint.
- Network isolation là quy tắc cốt lõi của thiết kế, vì sơ đồ ghi rõ data tier không có public access.
- Giữ data tier ở trạng thái private làm giảm attack surface và hạn chế các đường kết nối ngoài ý muốn.
- Mọi tương tác với database phải đi qua application layer.
- Secret nên cung cấp connection detail để tránh hard-code hostname hay credential trong artifact của ứng dụng.

## Lưu ý về vận hành và bảo mật

- Chỉ mở quyền truy cập database từ security group của ứng dụng thay vì từ CIDR rộng.
- Cần kiểm tra định kỳ backup retention, snapshot policy và maintenance window.
- Nên bật mã hóa dữ liệu ở trạng thái lưu trữ và trong quá trình truyền tải, đặc biệt khi dữ liệu liên quan đến tài chính hoặc compliance.
- Cần test connection pool, timeout và retry logic bằng failover thật thay vì chỉ giả định chúng sẽ hoạt động.
- Nếu bài toán đọc nhiều xuất hiện về sau, nên bổ sung read replica hoặc reporting isolation một cách có chủ đích.

## Best practices

- Dùng parameter group phù hợp với engine và quy trình migration có kiểm soát.
- Tài liệu hóa recovery expectation, bao gồm failover time điển hình và hành vi reconnect của ứng dụng.
- Kiểm tra point-in-time recovery và restore thực tế thay vì chỉ tin vào cấu hình.
- Lưu connection metadata trong Secrets Manager và xoay vòng credential theo policy của môi trường.

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
- Tưởng rằng relational persistence sẽ tự scale mãi mãi mà không cần tối ưu query hay tách tải đọc về sau.

## Ghi chú / Giả định

- Sơ đồ cho thấy rất rõ mô hình primary và standby kiểu Multi-AZ, nhưng không nêu chính xác engine hay deployment mode của RDS.
- Backup retention, point-in-time recovery và encryption setting không hiển thị trong hình.
- Không có read replica, cache hay analytics database trong sơ đồ nên không được giả định là đang tồn tại.
- DynamoDB và Aurora được coi là ngoài phạm vi của kiến trúc hiện tại.