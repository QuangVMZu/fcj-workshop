---
title: "Truy cập vận hành và quản trị bí mật"
date: 2026-04-04
weight: 5
chapter: false
pre: " <b> 4.5. </b> "
---

# Thực hành truy cập vận hành và quản trị bí mật

## Tổng quan

- Phần này giải thích cách operator và application instance truy cập môi trường mà không cần public server.
- Nội dung tập trung vào AWS Systems Manager, IAM instance profile, AWS Secrets Manager và AWS KMS.
- Đây là phần nên xem trước khi xử lý tình huống admin không vào được private EC2 hoặc ứng dụng không đọc được secret.

## Các dịch vụ AWS trong phần này

### AWS Systems Manager (SSM)
![AWS Systems Manager Session Manager](/images/4-Workshop/ssm-session-manager.png)
- Cung cấp truy cập có kiểm soát cho operator vào EC2 trong private subnet.
- Không cần mở SSH hay public IP.

### IAM Instance Profile
![IAM Instance Profile](/images/4-Workshop/iam-instance-profile.png)
- Cấp danh tính cho EC2 instance.
- Cho phép ứng dụng truy cập AWS service mà không cần hardcode credential.

### AWS Secrets Manager
- Lưu trữ các thông tin nhạy cảm như connection string, API key.
- Cho phép ứng dụng lấy secret một cách an toàn trong runtime.

### AWS KMS
- Mã hóa và bảo vệ secret.
- Kiểm soát quyền truy cập key thông qua IAM.

### Amazon EC2
![EC2 Instance IAM Role](/images/4-Workshop/ec2-iam-role.png)
- Sử dụng IAM role và lấy secret trong runtime.
- Là nơi thực thi application một cách an toàn.

## Vì sao thiết kế này dùng SSM và IAM

- Sơ đồ thể hiện admin access đi qua Systems Manager thay vì SSH public qua internet.
- Các EC2 instance dùng IAM role gắn kèm, nên ứng dụng không cần giữ AWS key dài hạn trong file.
- Đường truy cập được thể hiện là truy cập vận hành cho admin, không phải xác thực người dùng cuối. Cognito không xuất hiện trong thiết kế này.

## Luồng AWS từng bước

1. Operator cần shell access hoặc run-command access vào một EC2 instance private.
   Dịch vụ: AWS Systems Manager.
   Điều xảy ra: Operator khởi tạo một phiên SSM tới instance mục tiêu.
   Tại sao quan trọng: Không cần bastion host hay mở public SSH.
2. Instance xác thực bằng danh tính máy.
   Dịch vụ: IAM instance profile.
   Điều xảy ra: EC2 nhận temporary credential từ IAM role được gắn.
   Tại sao quan trọng: Ứng dụng có thể gọi AWS API mà không cần static access key.
3. Ứng dụng yêu cầu một secret.
   Dịch vụ: AWS Secrets Manager.
   Điều xảy ra: App gọi lấy secret theo tên trong runtime.
   Tại sao quan trọng: Credential database và token tích hợp có thể được xoay vòng ngoài gói triển khai.
4. Secret được giải mã theo mô hình bảo vệ của KMS.
   Dịch vụ: AWS KMS.
   Điều xảy ra: Secrets Manager dùng cơ chế mã hóa dựa trên KMS cho giá trị secret lưu trữ.
   Tại sao quan trọng: Secret được mã hóa và việc dùng khóa có thể được kiểm soát.
5. IAM role đó cũng có thể cho phép ghi log và gửi email.
   Dịch vụ: IAM, CloudWatch, SES.
   Điều xảy ra: Role của instance cho phép các thao tác AWS khác mà workload cần.
   Tại sao quan trọng: Một danh tính máy duy nhất có thể phục vụ nhiều tích hợp nhưng vẫn giữ được kiểm soát quyền.

## AWS CLI Walkthrough

### 1. Mở phiên SSM vào private instance

```bash
aws ssm start-session \
  --target ${INSTANCE_ID} \
  --region ${AWS_REGION}
```

### 2. Xác minh IAM instance profile đang được gắn

```bash
aws ec2 describe-instances \
  --instance-ids ${INSTANCE_ID} \
  --region ${AWS_REGION} \
  --query 'Reservations[0].Instances[0].IamInstanceProfile.Arn' \
  --output text
```

### 3. Đọc giá trị secret

```bash
aws secretsmanager get-secret-value \
  --secret-id ${DB_SECRET_ID} \
  --region ${AWS_REGION}
```

### 4. Liệt kê alias KMS để tìm khóa mã hóa secret

```bash
aws kms list-aliases --region ${AWS_REGION}
```

## Ví dụ cấu hình

### Quyền tối thiểu cho instance-role trong kiến trúc này

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue",
        "kms:Decrypt",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "ses:SendEmail",
        "ses:SendRawEmail"
      ],
      "Resource": "*"
    }
  ]
}
```

Ý nghĩa của policy này:

- Instance đọc được secret.
- Instance giải mã được dữ liệu thông qua KMS.
- Instance ghi log và gửi email được.
- Không cần lưu static AWS key trên máy chủ.

## Những gì cần xác minh

- EC2 instance mục tiêu đã được đăng ký trong Systems Manager.
- Instance có IAM instance profile đi kèm.
- Ứng dụng đọc được đúng các secret name mà nó cần.
- Quyền KMS chỉ cấp cho principal thực sự cần giải mã.
- Operator dùng SSM thay vì mở inbound SSH từ internet.

## Lỗi thường gặp

- Gắn role cho EC2 nhưng quên cấp `secretsmanager:GetSecretValue`.
- Có quyền Secrets Manager nhưng thiếu `kms:Decrypt` khi dùng customer-managed KMS key.
- Nhầm rằng xác thực người dùng cuối nằm ở đây trong khi sơ đồ chỉ thể hiện đường truy cập cho operator.
- Quay lại hard-coded credential vì naming convention của secret chưa rõ ràng.

## Ghi chú / Giả định

- Sơ đồ không ghi cụ thể chế độ SSM nào, nhưng Session Manager là cách hiểu trực tiếp nhất cho truy cập tương tác.
- Luồng xoay vòng secret không được thể hiện, nên workshop này tập trung vào retrieval và thiết kế quyền.
- Cognito và các dịch vụ định danh người dùng khác không xuất hiện trong sơ đồ kiến trúc.


