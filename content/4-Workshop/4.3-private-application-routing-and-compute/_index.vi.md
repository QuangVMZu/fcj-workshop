---
title: "Điều phối ứng dụng riêng tư và tầng tính toán"
date: 2026-04-04
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Thực hành điều phối ứng dụng riêng tư và tầng tính toán

## Tổng quan

- Phần này giải thích cách request động đi từ lớp edge vào compute private.
- Nội dung tập trung vào VPC, public và private subnet, Internet Gateway, NAT Gateway, ALB, EC2 instance và Auto Scaling.
- Đây là phần quan trọng nhất khi xử lý lỗi `5xx`, target unhealthy hoặc sự cố outbound connectivity.

## Các dịch vụ AWS trong phần này

### Amazon VPC
![Amazon VPC](/images/4-Workshop/vpc-overview.png)
- Định nghĩa ranh giới mạng cho toàn bộ hệ thống.
- Là nơi chứa tất cả subnet, gateway và compute resource.

### Internet Gateway
![Internet Gateway](/images/4-Workshop/internet-gateway.png)
- Gắn VPC với internet công khai.
- Cho phép traffic đi vào/ra khỏi VPC theo route table.

### NAT Gateway
![NAT Gateway](/images/4-Workshop/nat-gateway.png)
- Cho phép các instance trong private subnet truy cập internet.
- Không cho phép inbound traffic trực tiếp từ internet.

### Application Load Balancer
![Application Load Balancer](/images/4-Workshop/alb.png)
- Nhận request HTTP/HTTPS từ bên ngoài.
- Phân phối traffic tới các EC2 instance phía sau.

### Amazon EC2
![Amazon EC2 Instances](/images/4-Workshop/ec2-instances.png)
- Chạy application backend.
- Xử lý business logic và trả response.

### EC2 Auto Scaling
![Auto Scaling Group](/images/4-Workshop/auto-scaling-group.png)
- Tự động scale số lượng instance theo load.
- Đảm bảo high availability trên nhiều Availability Zone.

## Vì sao thiết kế này dùng ALB + EC2

- Sơ đồ thể hiện các application server chạy lâu dài, nên **Amazon EC2** là mô hình compute đang dùng.
- Điểm vào công khai của API là **Application Load Balancer**, không phải API Gateway.
- Không có Lambda, ECS hay EKS trong sơ đồ, nên workshop này không giả định serverless hay container orchestration.
- Việc đặt compute trong private subnet cho thấy instance không nên bị truy cập trực tiếp từ internet.

## Luồng AWS từng bước

1. Frontend gửi request động tới hostname của backend.
   Dịch vụ: Application Load Balancer.
   Điều xảy ra: ALB nhận request tại ranh giới công khai của tầng ứng dụng.
   Tại sao quan trọng: Chỉ ALB nên là thành phần public cho backend traffic.
2. ALB đánh giá listener rule và trạng thái health của target group.
   Dịch vụ: Application Load Balancer.
   Điều xảy ra: Request chỉ được chuyển tới target đã đăng ký và đang healthy.
   Tại sao quan trọng: Instance lỗi có thể bị loại ra mà không làm sập toàn hệ thống.
3. Một EC2 instance khỏe mạnh trong private subnet xử lý request.
   Dịch vụ: Amazon EC2.
   Điều xảy ra: Ứng dụng thực thi business logic và tạo response.
   Tại sao quan trọng: Compute vẫn private và được bảo vệ bằng security group.
4. Auto Scaling duy trì số lượng instance mong muốn.
   Dịch vụ: EC2 Auto Scaling.
   Điều xảy ra: Instance lỗi có thể được thay thế và công suất có thể tăng khi traffic tăng.
   Tại sao quan trọng: Nền tảng có thể tự phục hồi khi lỗi instance và scale ngang khi cần.
5. Nếu instance cần đi ra ngoài, traffic sẽ qua NAT Gateway.
   Dịch vụ: NAT Gateway và Internet Gateway.
   Điều xảy ra: Private instance truy cập endpoint bên ngoài mà không cần public IP.
   Tại sao quan trọng: Backend vẫn có thể gọi endpoint công khai của AWS hoặc tải dependency mà không public máy chủ.
6. Response đi ngược qua ALB về client.
   Dịch vụ: Application Load Balancer.
   Điều xảy ra: ALB hoàn tất vòng đời của HTTP request.
   Tại sao quan trọng: Client không bao giờ giao tiếp trực tiếp với EC2 instance.

## Logic nội bộ và hành vi request

- ALB là ranh giới HTTP công khai của backend. Nó quyết định request nào được vào fleet của ứng dụng và target nào đủ healthy để nhận traffic.
- EC2 instance là runtime chạy lâu dài của ứng dụng. Chúng được kỳ vọng xử lý request, duy trì connection pool và dùng private network cho các dependency downstream.
- Auto Scaling quản lý kích thước của fleet, còn ALB quản lý việc route request. Hai cơ chế này liên quan với nhau nhưng không giải quyết cùng một bài toán.
- NAT Gateway xử lý egress có kiểm soát cho private instance. Nó không nằm trên inbound request path, nhưng thường là thành phần rất quan trọng cho runtime dependency access.
- Vì kiến trúc hiện tại không cho thấy API Gateway, Lambda, ECS hay EKS, mô hình vận hành ở đây được giữ theo hướng instance-centric và network-centric.

## AWS CLI Walkthrough

### 1. Kiểm tra load balancer

```bash
aws elbv2 describe-load-balancers \
  --names ${ALB_NAME} \
  --region ${AWS_REGION}
```

### 2. Kiểm tra trạng thái target

```bash
aws elbv2 describe-target-health \
  --target-group-arn ${TARGET_GROUP_ARN} \
  --region ${AWS_REGION}
```

### 3. Kiểm tra Auto Scaling group

```bash
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names ${ASG_NAME} \
  --region ${AWS_REGION}
```

### 4. Kiểm tra một backend instance

```bash
aws ec2 describe-instances \
  --instance-ids ${INSTANCE_ID} \
  --region ${AWS_REGION}
```

## Ví dụ cấu hình

### Bộ rule traffic điển hình cho kiến trúc này

| Thành phần | Inbound rule | Outbound rule | Mục đích |
| --- | --- | --- | --- |
| Security group của ALB | `443` từ `0.0.0.0/0` | `${APP_PORT}` tới security group của EC2 | Điểm vào HTTPS công khai |
| Security group của EC2 | `${APP_PORT}` từ security group của ALB | Port database tới RDS, `443` cho AWS call outbound | Runtime private của ứng dụng |
| Route table của private subnet | Không có internet route trực tiếp | `0.0.0.0/0` tới NAT Gateway | Egress có kiểm soát |
| Route table của public subnet | `0.0.0.0/0` tới Internet Gateway | Đường công khai | Hỗ trợ ALB và NAT Gateway |

### Thiết lập health check ALB được khuyến nghị

- Protocol: `HTTP` hoặc `HTTPS`
- Health check path: endpoint nhẹ như `/actuator/health` hoặc `/health`
- Success codes: `200-399`
- Interval: `30` giây
- Healthy threshold: `2` trở lên
- Unhealthy threshold: `2` trở lên

## Lưu ý về scale và bảo mật

- Hãy giữ backend node càng stateless càng tốt để Auto Scaling có thể thay thế chúng mà không cần phục hồi session phức tạp.
- Chỉ cho phép inbound vào EC2 từ security group của ALB, thay vì mở application port cho CIDR public.
- Hãy xem NAT là dependency egress có kiểm soát. Nếu outbound availability trở nên quan trọng hơn, nên cân nhắc NAT Gateway theo từng Availability Zone hoặc VPC endpoint cho một số AWS service.
- Health check nên xác minh liveness của ứng dụng nhưng không được phụ thuộc quá nặng vào downstream service có thể lỗi độc lập.
- Nếu sau này nền tảng cần governance cho API ở mức cao hơn, API Gateway có thể được thêm vào trước một số endpoint cụ thể, nhưng đó là phần mở rộng kiến trúc chứ không phải sửa sai mô hình hiện tại.

## Best practices

- Dùng launch template và immutable deployment artifact để việc thay fleet được nhất quán.
- Trải Auto Scaling group trên nhiều Availability Zone.
- Giữ security group đơn giản và tách theo từng service để trust boundary rõ ràng.
- Theo dõi target health cùng với scaling activity, vì fleet đang healthy nhưng scale sai vẫn có thể làm hệ thống thất bại khi tăng tải.

## Những gì cần xác minh

- ALB là internet-facing, còn EC2 instance nằm trong private subnet.
- Target health là `healthy` cho mọi instance đang hoạt động.
- Auto Scaling group trải trên ít nhất hai Availability Zone.
- Private subnet dùng route qua NAT cho outbound traffic.
- Backend vẫn phản hồi bình thường dù instance không có public IP.

## Lỗi thường gặp

- Mở security group của EC2 ra internet thay vì chỉ cho ALB truy cập.
- Dùng health check path phụ thuộc vào database, làm instance bị đánh dấu lỗi khi database có sự cố.
- Quên rằng private instance vẫn cần outbound connectivity qua NAT.
- Tưởng rằng API Gateway hoặc Lambda tham gia vào flow trong khi điểm vào thật là ALB.

## Ghi chú / Giả định

- Sơ đồ chỉ thể hiện một NAT Gateway. Trong production, có thể cần một NAT Gateway cho mỗi Availability Zone để tăng độ bền outbound.
- Security group và NACL không được vẽ trên sơ đồ, nhưng là thành phần rất quan trọng để mô hình routing này hoạt động.
- Workshop này mô tả triển khai trên EC2 vì kiến trúc hiện tại không cho thấy ECS hay EKS.


