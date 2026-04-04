---
title: "Phân phối biên và yêu cầu dữ liệu bản đồ"
date: 2026-04-04
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# Thực hành phân phối biên và yêu cầu dữ liệu bản đồ

## Tổng quan

- Phần này tập trung vào lớp edge hướng công khai của nền tảng.
- Nó giải thích cách DNS, CDN caching, WAF filtering, S3 origin access và request tới Location Service hoạt động cùng nhau.
- Đây là điểm bắt đầu phù hợp khi frontend chậm, domain không resolve hoặc dữ liệu bản đồ bị thiếu.

## Các dịch vụ AWS trong phần này

### Amazon Route 53
![Amazon Route 53 DNS](/images/4-Workshop/route53.jpg)
- Phân giải domain công khai và chuyển người dùng đến điểm truy cập phù hợp.
- Là bước đầu tiên trong luồng request.

### Amazon CloudFront
![Amazon CloudFront Distribution](/images/4-Workshop/cloudfront.jpg)
- Phân phối frontend asset đã cache với độ trễ thấp.
- Giảm số lần truy vấn về origin (S3).

### AWS WAF
- Lọc các request HTTP/HTTPS đầu vào.
- Bảo vệ hệ thống trước các tấn công phổ biến ngay tại edge.

### Amazon S3
- Lưu trữ các file frontend (HTML, JS, CSS).
- Là origin cho CloudFront thông qua cơ chế Origin Access Control.

### Amazon Location Service
![Amazon Location Service](/images/4-Workshop/location-service.png)
- Xử lý các yêu cầu liên quan đến bản đồ và dữ liệu địa lý.
- Hoạt động tách biệt với luồng phân phối static asset.

## Luồng AWS từng bước

1. Người dùng mở `${APP_DOMAIN}` trên trình duyệt.
   Dịch vụ: Amazon Route 53.
   Điều xảy ra: Route 53 phân giải hostname công khai tới CloudFront distribution.
   Tại sao quan trọng: Nếu DNS sai, các dịch vụ AWS phía sau sẽ không được truy cập đúng.
2. CloudFront nhận HTTPS request.
   Dịch vụ: Amazon CloudFront và AWS WAF.
   Điều xảy ra: WAF rule kiểm tra request trước khi distribution phục vụ nội dung.
   Tại sao quan trọng: Traffic xấu hoặc bất thường có thể bị chặn ngay tại edge.
3. CloudFront kiểm tra file có nằm trong cache hay không.
   Dịch vụ: Amazon CloudFront và Amazon S3.
   Điều xảy ra: Nếu có cache thì trả ngay; nếu cache miss thì lấy từ S3.
   Tại sao quan trọng: Cache hit giúp giảm độ trễ và giảm tải cho origin bucket.
4. S3 chỉ trả object thông qua đường CloudFront đã được cho phép.
   Dịch vụ: Amazon S3 và CloudFront Origin Access Control.
   Điều xảy ra: Bucket vẫn private, còn CloudFront là bên đọc được ủy quyền.
   Tại sao quan trọng: Người dùng không thể bỏ qua CloudFront để truy cập trực tiếp vào S3.
5. Frontend sau đó gửi request lấy dữ liệu địa lý.
   Dịch vụ: Amazon Location Service.
   Điều xảy ra: Trình duyệt hoặc mã frontend gọi place index và map API.
   Tại sao quan trọng: Luồng map được tách khỏi luồng phân phối static asset.
6. Frontend tiếp tục gọi backend API qua tầng ứng dụng.
   Dịch vụ: Application Load Balancer ở phần tiếp theo.
   Điều xảy ra: Traffic động rời lớp edge và đi vào compute stack private.
   Tại sao quan trọng: Static traffic và dynamic traffic có thể được tối ưu riêng.

## AWS CLI Walkthrough

### 1. Kiểm tra hosted zone

```bash
aws route53 list-hosted-zones --region ${AWS_REGION}
```

### 2. Tạo hoặc cập nhật bản ghi DNS của ứng dụng

Tạo file `route53-app-record.json`:

```json
{
  "Comment": "Point app subdomain to CloudFront",
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "app.example.com",
        "Type": "CNAME",
        "TTL": 300,
        "ResourceRecords": [
          {
            "Value": "d111111abcdef8.cloudfront.net"
          }
        ]
      }
    }
  ]
}
```

Áp dụng thay đổi và chờ trạng thái đồng bộ:

```bash
CHANGE_ID=$(aws route53 change-resource-record-sets \
  --hosted-zone-id ${HOSTED_ZONE_ID} \
  --change-batch file://route53-app-record.json \
  --query 'ChangeInfo.Id' \
  --output text)

aws route53 wait resource-record-sets-changed --id ${CHANGE_ID}
```

### 3. Triển khai frontend asset lên S3

```bash
aws s3 sync ./dist s3://${STATIC_BUCKET} --delete
```

### 4. Xóa cache CloudFront sau khi deploy

```bash
aws cloudfront create-invalidation \
  --distribution-id ${CLOUDFRONT_DIST_ID} \
  --paths "/*"
```

### 5. Thử một truy vấn tìm kiếm với Location Service

```bash
aws location search-place-index-for-text \
  --index-name ${PLACE_INDEX_NAME} \
  --text "EV charging station" \
  --region ${AWS_REGION}
```

## Ví dụ cấu hình

### S3 bucket policy private cho CloudFront OAC

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipalReadOnly",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::ev-prod-frontend/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/E123ABC456DEF"
        }
      }
    }
  ]
}
```

Ý nghĩa của cấu hình này:

- Bucket S3 vẫn giữ trạng thái private.
- Chỉ CloudFront distribution được chỉ định mới có quyền đọc object.
- Người dùng buộc phải đi qua CloudFront, nơi WAF có thể kiểm tra traffic.

## Những gì cần xác minh

- DNS resolve về đúng CloudFront distribution.
- Bucket S3 không public read.
- Sau mỗi lần deploy đều có CloudFront invalidation.
- Chức năng tìm kiếm bản đồ hoạt động qua Amazon Location Service.
- WAF đã được gắn với CloudFront distribution trong môi trường mục tiêu.

## Lỗi thường gặp

- Cập nhật nội dung S3 nhưng quên invalidate CloudFront.
- Mở public bucket S3 thay vì dùng OAC.
- Trỏ Route 53 đến sai CloudFront domain.
- Chẩn đoán lỗi map ở backend trong khi nguyên nhân thật nằm ở cấu hình Amazon Location.

## Ghi chú / Giả định

- Sơ đồ thể hiện Amazon Location Service là đường tích hợp cho frontend, nhưng không ghi rõ map style hay place index cụ thể.
- Trong môi trường production, Route 53 có thể dùng alias thay vì CNAME. Ví dụ ở đây dùng CNAME để dễ hiểu.
- Sơ đồ không có API Gateway hay Lambda ở lớp edge. Request động được chuyển xuống tầng ứng dụng dùng ALB.


