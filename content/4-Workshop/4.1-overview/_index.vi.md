---
title: "Tổng quan"
date: 2026-04-04
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Tổng quan workshop và bản đồ dịch vụ AWS

![AWS Workshop Architecture Overview](/images/4-Workshop/ev-achi.png)

## Tổng quan hệ thống

- EV Charging Station Management System được triển khai như một nền tảng web nhiều lớp trên AWS.
- Người dùng truy cập frontend qua Route 53 và CloudFront.
- Static asset được lưu trong Amazon S3 và được bảo vệ phía sau CloudFront Origin Access Control.
- Request backend được chuyển tới các EC2 instance private thông qua Application Load Balancer.
- Dữ liệu nghiệp vụ được lưu trong Amazon RDS và không public ra bên ngoài.
- Đội vận hành sử dụng các dịch vụ gốc của AWS như Systems Manager, Secrets Manager, CloudWatch, CloudTrail và SES.

## Bản đồ dịch vụ AWS

| Nhu cầu kiến trúc | Dịch vụ dùng trong thiết kế này | Lý do sử dụng | Không dùng ở đây |
| --- | --- | --- | --- |
| DNS công khai | Amazon Route 53 | Điều hướng người dùng tới điểm vào của hệ thống | API Gateway custom domain |
| Phân phối frontend | Amazon CloudFront + Amazon S3 | Phát static asset toàn cầu với cache | Amplify Hosting |
| Bảo vệ edge | AWS WAF | Lọc web traffic không mong muốn | Chi tiết cấu hình Shield không được thể hiện |
| Tra cứu bản đồ | Amazon Location Service | Phục vụ tìm kiếm địa lý và tính năng bản đồ | Nhà cung cấp bản đồ bên thứ ba |
| Điểm vào API | Application Load Balancer | Chuyển tiếp HTTP traffic tới compute private | API Gateway |
| Compute | Amazon EC2 + Auto Scaling | Chạy backend process lâu dài | Lambda, ECS, EKS |
| Outbound internet | NAT Gateway + Internet Gateway | Cho phép instance private đi ra ngoài khi cần | EC2 public |
| Lưu trữ quan hệ | Amazon RDS | Lưu dữ liệu giao dịch với standby support | DynamoDB |
| Truy cập admin | AWS Systems Manager | Tránh phải mở SSH trực tiếp | Bastion host |
| Quản lý secret | AWS Secrets Manager + AWS KMS | Tập trung hóa và mã hóa secret | Hard-coded credentials |
| Monitoring | Amazon CloudWatch | Thu thập metrics và logs | APM bên thứ ba không được thể hiện |
| Audit | AWS CloudTrail | Ghi nhận hoạt động API của AWS | Fan-out audit bằng EventBridge |
| Email | Amazon SES | Gửi email giao dịch | SNS email delivery |

## Luồng AWS đầu cuối

1. Người dùng phân giải `${APP_DOMAIN}` thông qua Amazon Route 53.
   Dịch vụ: Amazon Route 53.
   Tại sao quan trọng: DNS là điểm kiểm soát đầu tiên của tên miền công khai.
2. CloudFront nhận request và đánh giá rule của AWS WAF.
   Dịch vụ: Amazon CloudFront và AWS WAF.
   Tại sao quan trọng: Lớp edge giảm độ trễ và chặn request xấu trước khi chạm backend.
3. CloudFront trả nội dung từ cache hoặc lấy từ Amazon S3.
   Dịch vụ: Amazon CloudFront và Amazon S3.
   Tại sao quan trọng: Frontend được phát nhanh, còn bucket S3 không cần public.
4. Frontend gọi backend API thông qua Application Load Balancer.
   Dịch vụ: Application Load Balancer.
   Tại sao quan trọng: ALB là điểm vào HTTP công khai cho request động.
5. ALB chuyển traffic tới các EC2 instance đang healthy trong private subnet.
   Dịch vụ: ALB, Amazon EC2, EC2 Auto Scaling.
   Tại sao quan trọng: Backend có thể scale ngang mà không phải public instance.
6. Ứng dụng trên EC2 đọc và ghi dữ liệu quan hệ trong Amazon RDS.
   Dịch vụ: Amazon RDS.
   Tại sao quan trọng: Giao dịch được lưu trong dịch vụ quan hệ được quản lý và có standby.
7. Ứng dụng lấy secret, ghi log và gửi email bằng IAM role của instance.
   Dịch vụ: IAM, Secrets Manager, KMS, CloudWatch, SES.
   Tại sao quan trọng: Không cần lưu static credential trên máy chủ.
8. Đội vận hành quản trị môi trường qua Systems Manager và kiểm toán bằng CloudTrail.
   Dịch vụ: AWS Systems Manager và AWS CloudTrail.
   Tại sao quan trọng: Hoạt động quản trị có thể truy vết và không cần public SSH.

## Các bước kiểm tra nền tảng

Hãy chạy các lệnh sau trước khi đi vào những phần còn lại.

```bash
aws sts get-caller-identity
aws route53 list-hosted-zones --region ${AWS_REGION}
aws cloudfront list-distributions --region ${AWS_REGION}
aws ec2 describe-instances --region ${AWS_REGION} --max-items 5
aws rds describe-db-instances --region ${AWS_REGION}
```

Những điều cần kiểm tra:

- Bạn đang xác thực đúng AWS account mong muốn.
- Hosted zone, CloudFront distribution, EC2 instance và RDS instance đều tồn tại trong môi trường đúng.
- Region và naming convention khớp với các giá trị bạn sẽ dùng ở các lab tiếp theo.

## Hướng dẫn cho người mới bắt đầu

- Hãy bắt đầu bằng cách phân biệt thành phần nào public và thành phần nào private.
- Xem Route 53, CloudFront, ALB và SES là các dịch vụ hướng công khai.
- Xem EC2, RDS, Secrets Manager và KMS là các dịch vụ nội bộ cần kiểm soát chặt.
- Xem Systems Manager, CloudWatch và CloudTrail là bộ công cụ cho operator.
- Nếu một dịch vụ không xuất hiện trong sơ đồ, không nên tự giả định rằng nó tồn tại trong môi trường triển khai.

## Ghi chú / Giả định

- Sơ đồ thể hiện topology hạ tầng, không phải source code ứng dụng hay pipeline CI/CD.
- Proposal hiện có nhắc đến Amazon RDS for SQL Server, nhưng bản thân sơ đồ chỉ ghi Amazon RDS. Hãy điều chỉnh lệnh theo engine thực tế.
- Không có thành phần API Gateway, Lambda, ECS, EKS, DynamoDB, EventBridge, SQS, SNS hay Cognito nào xuất hiện trong kiến trúc này.


