---
title: "Workshop"
date: 2026-04-04
weight: 4
chapter: false
pre: " <b> 4. </b> "
---

# Workshop thực hành AWS cho nền tảng quản lý trạm sạc xe điện

![Tong quan kien truc workshop AWS](/images/2-Proposal/AWS-architecture.png)

## Tổng quan

Chương này là tài liệu workshop thực hành cho kiến trúc AWS đứng sau EV Charging Station Management System. Tài liệu không hướng dẫn chạy ứng dụng local. Thay vào đó, nó giải thích cách các dịch vụ AWS trong môi trường triển khai nhận traffic, chạy backend, lưu dữ liệu, bảo vệ secret và hỗ trợ vận hành.

## Cách sử dụng workshop

- Đi theo đúng thứ tự các phần, từ edge services đến observability.
- Thay các giá trị mẫu như `${APP_DOMAIN}` và `${DB_INSTANCE_ID}` bằng thông tin của môi trường thật.
- Các ví dụ CLI dùng cú pháp `bash`. Nếu bạn dùng PowerShell, hãy điều chỉnh phần dấu nháy cho phù hợp.
- Nên thao tác trên môi trường không phải production khi thử Route 53, CloudFront invalidation hoặc RDS failover.
- Hãy đọc mỗi phần theo hai lượt: lượt đầu để hiểu flow, lượt sau để làm theo command và bước kiểm tra.

## Các dịch vụ được dùng trong kiến trúc này

- **Edge và phân phối frontend**: Amazon Route 53, Amazon CloudFront, AWS WAF, Amazon S3, Amazon Location Service
- **Tầng chạy ứng dụng**: Amazon VPC, Internet Gateway, NAT Gateway, Application Load Balancer, Amazon EC2, EC2 Auto Scaling
- **Tầng dữ liệu**: Amazon RDS với bố cục primary và standby
- **Vận hành và bảo mật**: AWS Systems Manager, IAM instance profile, AWS Secrets Manager, AWS KMS
- **Quan sát và giao tiếp**: Amazon CloudWatch, AWS CloudTrail, Amazon SES, các bản ghi DNS email trong Route 53

## Các dịch vụ không xuất hiện trong sơ đồ này

- **API Gateway**: Không dùng trong thiết kế này vì traffic công khai đi vào qua CloudFront rồi tới Application Load Balancer.
- **Lambda**: Không được thể hiện. Backend đang được mô hình hóa bằng các EC2 instance chạy lâu dài.
- **ECS / EKS**: Không được thể hiện. Container orchestration nằm ngoài phạm vi của kiến trúc hiện tại.
- **DynamoDB**: Không được thể hiện. Dữ liệu chính được mô hình hóa dưới dạng quan hệ trong Amazon RDS.
- **EventBridge / SQS / SNS**: Không được thể hiện. Sơ đồ tập trung vào luồng request đồng bộ và tích hợp trực tiếp giữa các dịch vụ.
- **Cognito**: Không được thể hiện. Đường truy cập admin đang dùng IAM và AWS Systems Manager.

## Chương này bao quát điều gì trong thực tế

- Nó giải thích cách một request từ trình duyệt đi vào AWS, tải frontend, rồi chuyển tiếp sang backend API.
- Nó làm rõ ranh giới giữa lớp public và các tầng application, data nằm trong private network.
- Nó cho thấy đội vận hành có thể vào môi trường mà không cần mở SSH ra internet.
- Nó nhấn mạnh các dịch vụ hỗ trợ khiến runtime đủ mức production như secret management, audit logging, metrics và outbound email.
- Nó cố ý tập trung vào kiến trúc AWS đã triển khai để người đọc hiểu được runtime behavior, failure domain và trách nhiệm vận hành theo từng lớp.

## Biến môi trường dùng chung trong lab

```bash
export AWS_REGION=ap-southeast-1
export APP_DOMAIN=app.example.com
export HOSTED_ZONE_ID=Z123456789EXAMPLE
export STATIC_BUCKET=ev-prod-frontend
export CLOUDFRONT_DIST_ID=E123ABC456DEF
export PLACE_INDEX_NAME=ev-place-index
export ALB_NAME=ev-app-alb
export TARGET_GROUP_ARN=arn:aws:elasticloadbalancing:ap-southeast-1:123456789012:targetgroup/ev-app-tg/abc123
export ASG_NAME=ev-app-asg
export INSTANCE_ID=i-0123456789abcdef0
export APP_PORT=8080
export DB_INSTANCE_ID=ev-prod-rds
export DB_PORT=1433
export DB_SECRET_ID=ev/prod/database
export LOG_GROUP=/aws/ec2/ev-app
export SES_IDENTITY=example.com
export SES_SENDER=no-reply@example.com
export SES_RECIPIENT=ops@example.com
```

## Quyền AWS nên có

- Quyền đọc và thay đổi Route 53 cho các bước xác thực DNS
- Quyền với CloudFront và S3 để kiểm tra luồng phân phối frontend
- Quyền đọc EC2, ELBv2, Auto Scaling và VPC để kiểm tra tầng ứng dụng
- Quyền đọc RDS, và chỉ cấp quyền reboot có kiểm soát ở môi trường được cho phép
- Quyền với SSM, Secrets Manager, IAM, KMS, CloudWatch, CloudTrail và SES để thao tác vận hành

## Nội dung

1. [Tổng quan](4.1-overview/)
2. [Phân phối biên và yêu cầu dữ liệu bản đồ](4.2-edge-delivery-and-geospatial-requests/)
3. [Điều phối ứng dụng riêng tư và tầng tính toán](4.3-private-application-routing-and-compute/)
4. [Độ bền dữ liệu quan hệ và cô lập mạng](4.4-relational-data-resilience-and-network-isolation/)
5. [Truy cập vận hành và quản trị bí mật](4.5-operational-access-and-secret-governance/)
6. [Giám sát, kiểm toán và gửi email](4.6-monitoring-audit-and-email-delivery/)
7. [Phân tích kiến trúc workshop chuyên sâu](4.7-workshop-architecture-deep-dive/)

