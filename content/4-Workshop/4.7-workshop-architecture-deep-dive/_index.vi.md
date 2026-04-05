---
title: "Phân tích kiến trúc workshop chuyên sâu"
date: 2026-04-05
weight: 7
chapter: false
pre: " <b> 4.7. </b> "
---

# Phân tích kiến trúc theo chuẩn production cho workshop AWS

![Tong quan kien truc workshop AWS](/images/2-Proposal/AWS-architecture.png)

## Tổng quan

- Tài liệu này tổng hợp các source hiện có nằm dưới `content/4-Workshop/` và gom chúng lại thành một tài liệu kiến trúc hoàn chỉnh ở mức production.
- Phần `4-Workshop` là codebase tài liệu chứ không phải code ứng dụng, vì vậy các "module" bên dưới được hiểu là các module năng lực kiến trúc được suy ra từ các mục `4.1` đến `4.6`.
- Mô hình runtime hiện tại được thể hiện rất nhất quán: Route 53 và CloudFront tạo thành lớp edge công khai, Amazon S3 lưu frontend tĩnh, backend chạy trên EC2 private phía sau Application Load Balancer, Amazon RDS là hệ thống lưu trữ quan hệ trung tâm, còn các dịch vụ gốc của AWS phụ trách vận hành, bí mật, giám sát, kiểm toán và email.
- Ở những chỗ workshop chưa diễn đạt hết chi tiết triển khai, tài liệu này bổ sung các giả định hợp lý để kiến trúc trở thành một thiết kế dịch vụ AWS đầy đủ và chuyên nghiệp hơn.

## Cách sử dụng phần phân tích chuyên sâu này

- Nên đọc phần này sau `4.1` đến `4.6` để mạch kiến trúc khớp với đúng thứ tự thực hành của workshop.
- Có thể dùng phần này trong design review, onboarding, troubleshooting hoặc release-readiness check khi nhóm cần một góc nhìn kiến trúc thống nhất.
- Hãy xem những dịch vụ được vẽ trong sơ đồ hiện có là phần đã được xác nhận của thiết kế, còn các kiểm soát bổ sung trong tài liệu này là các thành phần hỗ trợ được suy ra hợp lý từ chính workshop.
- Khi tài liệu nhắc tới các dịch vụ tương lai như API Gateway, Lambda, Step Functions, EventBridge, SQS hoặc SNS, đó là hướng mở rộng chứ không phải khẳng định rằng stack hiện tại đã triển khai chúng.

## Phạm vi nguồn được phân tích

| Nguồn workshop đã đọc | Vai trò trong chương | Tín hiệu kiến trúc rút ra |
| --- | --- | --- |
| `4-Workshop/_index.md` và `4-Workshop/_index.vi.md` | Định nghĩa phạm vi chương, biến dùng chung, quyền truy cập khuyến nghị và các dịch vụ nằm trong hoặc ngoài phạm vi | Xác lập ranh giới chuẩn của workshop và xác nhận rằng ALB, EC2, RDS, SSM, Secrets Manager, KMS, CloudWatch, CloudTrail và SES là các khối AWS chính |
| `4-Workshop/4.1-overview/_index.md` và `4-Workshop/4.1-overview/_index.vi.md` | Giới thiệu bản đồ dịch vụ và luồng request đầu cuối | Xác nhận kiến trúc nhiều lớp và ranh giới giữa thành phần public và private |
| `4-Workshop/4.2-edge-delivery-and-geospatial-requests/_index.md` và `4-Workshop/4.2-edge-delivery-and-geospatial-requests/_index.vi.md` | Mô tả DNS, CDN, WAF, bảo vệ origin S3 và Amazon Location Service | Xác định module edge, đường đi của static content và luồng tích hợp bản đồ |
| `4-Workshop/4.3-private-application-routing-and-compute/_index.md` và `4-Workshop/4.3-private-application-routing-and-compute/_index.vi.md` | Mô tả VPC, ALB, EC2, private subnet và Auto Scaling | Xác định đường thực thi API đồng bộ và mô hình cô lập mạng cho compute |
| `4-Workshop/4.4-relational-data-resilience-and-network-isolation/_index.md` và `4-Workshop/4.4-relational-data-resilience-and-network-isolation/_index.vi.md` | Mô tả Amazon RDS, cơ chế primary và standby, và việc không public database | Xác định mô hình lưu trữ giao dịch và các quy tắc cô lập tầng dữ liệu |
| `4-Workshop/4.5-operational-access-and-secret-governance/_index.md` và `4-Workshop/4.5-operational-access-and-secret-governance/_index.vi.md` | Mô tả truy cập vận hành, danh tính máy bằng IAM, đọc secret và mã hóa bằng KMS | Xác định control plane cho bảo mật và mô hình tiêu thụ secret |
| `4-Workshop/4.6-monitoring-audit-and-email-delivery/_index.md` và `4-Workshop/4.6-monitoring-audit-and-email-delivery/_index.vi.md` | Mô tả logging, audit, gửi mail bằng SES và các bản ghi DNS hỗ trợ | Xác định lớp observability, audit trail và đường gửi thông điệp ra ngoài |

## Đọc workshop như một codebase kiến trúc

- Chương workshop được tổ chức giống như một hệ module hạ tầng theo lớp.
- Mỗi subsection đưa ra một trách nhiệm nền tảng, các dịch vụ AWS liên quan, bộ lệnh thao tác và tiêu chí kiểm tra riêng.
- File index của chương đóng vai trò như một architecture manifest vì nó khai báo biến dùng chung, quyền truy cập và các dịch vụ cố ý không dùng.
- Các trang con đóng vai trò như module triển khai vì chúng mô tả hành vi, cung cấp lệnh CLI để kiểm tra hoặc thay đổi tài nguyên, đồng thời nêu rõ trạng thái nào được xem là đúng.
- Cây tiếng Anh và tiếng Việt là hai biến thể ngôn ngữ của cùng một thiết kế kỹ thuật, không phải hai kiến trúc khác nhau.

## Kiến trúc nền tảng

Workshop mô tả baseline runtime AWS như sau:

```text
User -> Route 53 -> CloudFront/WAF -> S3
Frontend -> Amazon Location Service
Frontend -> ALB -> Private EC2 -> Amazon RDS
Private EC2 -> Secrets Manager/KMS -> CloudWatch -> SES
Admin -> Systems Manager -> Private EC2
AWS control-plane activity -> CloudTrail
```

### Quan điểm thiết kế đã được xác nhận

- Public traffic chỉ dừng ở edge và application entry point, không đi thẳng vào EC2.
- Static traffic và dynamic traffic được tách riêng có chủ đích.
- Backend là workload chạy lâu dài trên instance, vì vậy EC2 là mô hình thực thi chính.
- Dữ liệu nghiệp vụ là dữ liệu quan hệ và được đặt ở vùng private, vì vậy Amazon RDS là mô hình lưu trữ chính.
- Truy cập vận hành đi theo đường AWS-native và tránh SSH public trực tiếp.
- Kiến trúc hiện tại chủ yếu là đồng bộ. Trong workshop chưa có hàng đợi, event bus hay state-machine orchestration được vẽ ra rõ ràng.

### Các thành phần bắt buộc được suy ra

Workshop không vẽ hết mọi thành phần cấp thấp của AWS, nhưng kiến trúc này chỉ hoạt động đúng khi có các khối sau:

| Thành phần được suy ra | Lý do bắt buộc phải có |
| --- | --- |
| Chứng chỉ trong AWS Certificate Manager | HTTPS trên CloudFront và ALB cần chứng chỉ TLS hợp lệ dù sơ đồ không ghi rõ ACM |
| Security group | Việc phân tách public và private chỉ thực sự có hiệu lực khi có rule cụ thể giữa CloudFront, ALB, EC2 và RDS |
| Route table và subnet association | Internet Gateway và NAT Gateway chỉ hoạt động đúng khi public subnet và private subnet có route table phù hợp |
| Launch template hoặc launch configuration | Auto Scaling Group cần biết AMI, instance type, security group và user data để tạo EC2 mới |
| RDS subnet group | Tầng database private trên nhiều subnet cần một DB subnet group rõ ràng |
| SSM Agent và cơ chế chuyển log lên CloudWatch | Quản trị EC2 private và thu log runtime thường phụ thuộc vào agent được cài và chạy ổn định |
| IAM trust policy và KMS key policy | Instance profile, giải mã secret và quyền gọi AWS API đều phụ thuộc vào policy đúng |

## Module lõi, dịch vụ, API và luồng dữ liệu

| Module lõi | Dịch vụ AWS trong phạm vi | Đầu vào chính | Đầu ra chính | Phụ thuộc trực tiếp |
| --- | --- | --- | --- | --- |
| Module phân phối edge | Route 53, CloudFront, AWS WAF, Amazon S3 | DNS lookup, HTTPS request lấy static asset | HTML, JavaScript, CSS, phản hồi đã cache | Hosted zone, CloudFront distribution, S3 bucket, OAC, TLS certificate |
| Module tích hợp dữ liệu bản đồ | Amazon Location Service | Search text, tọa độ, map tile request | Kết quả địa điểm, metadata bản đồ, phản hồi geospatial | Cấu hình frontend, mô hình ký quyền truy cập, khả dụng theo region |
| Module vào API | Application Load Balancer | HTTPS API request từ frontend | Request được route tới backend healthy | Listener rule, target group, health check, public subnet |
| Module runtime ứng dụng | VPC, private subnet, EC2, Auto Scaling, NAT Gateway, Internet Gateway | API request đã được route, cấu hình runtime, outbound call | Response nghiệp vụ, log, email, truy vấn database | AMI, instance profile, security group, route table, NAT egress |
| Module lưu trữ quan hệ | Amazon RDS với bố cục primary và standby | SQL read, write, transaction, reconnect event | Bản ghi bền vững, khả năng failover, private endpoint | DB subnet group, security group, cấu hình engine, backup |
| Module bảo mật và vận hành | Systems Manager, IAM instance profile, Secrets Manager, KMS | Yêu cầu vào máy, yêu cầu đọc secret, luồng xác thực AWS API | Temporary credential, giá trị secret đã giải mã, phiên shell có kiểm soát | Đăng ký SSM, IAM trust, quyền KMS, chuẩn đặt tên secret |
| Module quan sát và giao tiếp | CloudWatch, CloudTrail, SES, DNS email trong Route 53 | Log, metric, AWS API event, email send request | Tín hiệu vận hành, lịch sử audit, email outbound | IAM permission, SES identity đã verify, DNS record hỗ trợ |

## Danh mục giao diện và API

| Giao diện | Bên phát sinh | Bên tiếp nhận | Giao thức hoặc kiểu API | Mục đích |
| --- | --- | --- | --- | --- |
| Phân giải domain công khai | Trình duyệt | Route 53 | DNS | Phân giải domain của ứng dụng về CloudFront distribution |
| Phân phối frontend tĩnh | Trình duyệt | CloudFront và S3 | HTTPS GET | Trả SPA asset với caching và private origin |
| Bảo vệ edge | CloudFront | AWS WAF | Web ACL inspection | Lọc request xấu hoặc bất thường trước khi chạm origin |
| Tìm kiếm địa lý | Frontend | Amazon Location Service | HTTPS JSON API | Tìm kiếm trạm sạc và dữ liệu bản đồ |
| API ứng dụng động | Frontend | ALB tới EC2 | HTTPS JSON theo kiểu REST | Thực thi auth, booking, charging, payment và các nghiệp vụ vận hành |
| Phiên database | Ứng dụng trên EC2 | Amazon RDS | SQL qua port theo engine | Lưu và truy vấn dữ liệu giao dịch |
| Đọc secret | Ứng dụng trên EC2 | Secrets Manager | AWS control-plane API | Nạp credential database và cấu hình runtime |
| Giải mã secret | Secrets Manager | AWS KMS | Cơ chế dùng khóa được quản lý | Bảo vệ secret ở trạng thái lưu trữ và khi truy xuất hợp lệ |
| Truy cập operator | Quản trị viên | Systems Manager | Session Manager control channel | Truy cập EC2 private mà không cần bastion hay SSH public |
| Gửi log và metric | Ứng dụng hoặc agent trên EC2 | CloudWatch | API ingest log và metric | Lưu lại tín hiệu observability của runtime |
| Ghi nhận audit | Dịch vụ AWS | CloudTrail | Audit event do dịch vụ sinh ra | Theo dõi hoạt động quản trị và control-plane |
| Gửi email | Ứng dụng trên EC2 | Amazon SES | API kiểu `SendEmail` hoặc `SendRawEmail` | Gửi email thông báo hoặc email giao dịch |

## Ánh xạ dịch vụ AWS và hướng mở rộng kiến trúc

| Năng lực kiến trúc | Ánh xạ hiện tại trong workshop | Vì sao ánh xạ này phù hợp với thiết kế đang có | Hướng mở rộng ở mức production |
| --- | --- | --- | --- |
| Public DNS | Route 53 | Workshop dùng domain public và DNS là điểm chạm đầu tiên của mọi flow | Có thể bổ sung health check routing, weighted cutover hoặc record cho disaster recovery nếu sau này mở rộng multi-region |
| CDN và phân phối web tĩnh | CloudFront cộng với S3 | Frontend là static và hưởng lợi rõ rệt từ cache toàn cầu với origin private | Có thể siết chặt cache policy, response headers policy và signed URL nếu sau này cần bảo vệ nội dung cao hơn |
| Bảo mật edge | AWS WAF | Workshop đặt lớp lọc request ngay ở edge một cách rõ ràng | Có thể bổ sung managed rule group, IP reputation, rate-based rule và bot control |
| Công bố API public | Application Load Balancer | Backend đang chạy trên EC2 và workshop nêu rất rõ là hiện không dùng API Gateway | Nếu sau này cần quản trị API public ở mức sản phẩm, authorizer, request validation hoặc throttle theo client, có thể đưa API Gateway lên trước ALB hoặc đứng trước một nhóm endpoint riêng |
| Runtime compute | EC2 cộng với Auto Scaling | Workload chạy lâu dài và cơ chế thay thế theo health check phù hợp với instance-based compute | Nếu có những tác vụ nhỏ, chạy ngắn hoặc mang tính event-driven, có thể thêm Lambda cho utility function mà không cần thay backend chính |
| Lưu trữ object | Amazon S3 | Frontend tĩnh rất phù hợp với object storage và OAC của CloudFront | Có thể thêm versioning, lifecycle policy và prefix theo release để rollback dễ hơn |
| Dữ liệu quan hệ | Amazon RDS | Workshop mô tả dữ liệu giao dịch và bố cục primary cùng standby, rất phù hợp với RDS | Nếu nhu cầu đọc tăng mạnh hoặc cần tách workload, có thể cân nhắc read replica, Aurora hoặc database báo cáo riêng |
| Quản lý khóa và secret | Secrets Manager cộng với KMS | Workshop chủ đích tách credential ra khỏi code và mã hóa chúng | Có thể thêm secret rotation tự động, siết chặt resource scope và tách KMS key theo môi trường |
| Truy cập operator private | Systems Manager cộng với IAM | Phù hợp với yêu cầu không mở SSH public trong workshop | Có thể thêm session logging, phê duyệt just-in-time và runbook vận hành bằng document |
| Monitoring và audit | CloudWatch cộng với CloudTrail | Workshop giữ observability và control-plane audit hoàn toàn theo bộ công cụ native của AWS | Có thể thêm alarm, dashboard, retention policy, metric filter và phân tích bảo mật tập trung |
| Gửi email | Amazon SES cộng với Route 53 DNS | Ứng dụng cần email giao dịch và SES khớp với mô hình tích hợp trực tiếp đang được thể hiện | Có thể thêm bounce và complaint processing, sending domain chuyên biệt và suppression handling |
| Điều phối workflow | Chưa có trong workshop hiện tại | Luồng request hiển thị đang là đồng bộ và chưa cần state machine | Step Functions là lựa chọn tự nhiên nếu quy trình settlement, charging session hoặc compliance sau này trở nên nhiều bước và kéo dài |
| Eventing bất đồng bộ | Chưa có trong workshop hiện tại | Trên sơ đồ và trong nội dung chưa có đường EventBridge, SQS hay SNS | EventBridge, SQS và SNS là hướng mở rộng mạnh cho fan-out thông báo, retry isolation và xử lý nền sau giao dịch |
| NoSQL persistence | Chưa có trong workshop hiện tại | Hệ thống dữ liệu chính hiện là dữ liệu quan hệ nên DynamoDB đang ngoài phạm vi | DynamoDB chỉ thực sự hợp lý nếu sau này cần key-value latency thấp, idempotency store hoặc mô hình đọc đã được denormalize theo sự kiện |

## Phân rã chi tiết workshop

### Chương gốc 4-Workshop

Phần này làm gì:

- Thiết lập từ vựng chung và ranh giới dịch vụ cho toàn bộ chương.
- Khai báo các biến lab như `${APP_DOMAIN}`, `${STATIC_BUCKET}`, `${ALB_NAME}`, `${DB_INSTANCE_ID}` và `${SES_IDENTITY}`.
- Nói rõ dịch vụ nào không có trong thiết kế hiện tại để người đọc không tự thêm các thành phần mà workshop không hề mô tả.

Vì sao phần này tồn tại:

- Nó đóng vai trò như architecture manifest của cả workshop.
- Nó giữ cho các phần thực hành sau luôn thống nhất về region, identifier và kỳ vọng vận hành.
- Nó chặn scope drift bằng cách nêu rõ rằng API Gateway, Lambda, ECS, EKS, DynamoDB, EventBridge, SQS, SNS và Cognito không nằm trong triển khai hiện tại.

Hành vi đầu vào và đầu ra:

| Đầu vào hoặc kích hoạt | Xử lý trong chương gốc | Đầu ra hoặc tác dụng |
| --- | --- | --- |
| Người đọc đi vào chương workshop | Ngữ cảnh dùng chung được thiết lập | Người đọc biết dịch vụ nào đang nằm trong phạm vi và chương được tổ chức ra sao |
| Người đọc cần bối cảnh CLI | Danh sách biến môi trường được cung cấp | Các lệnh ở những phần sau có thể tái sử dụng một cách nhất quán |
| Người đọc hỏi hệ thống chưa dùng gì | Danh sách dịch vụ bị loại trừ được nêu rõ | Kiến trúc bám sát thiết kế đang hiển thị thay vì trôi sang các giả định khác |

Giải thích ở mức cao về các artifact trong workshop:

- Bộ biến môi trường dùng chung không phải application code mà là lớp tiện ích cho operator để mọi lệnh sau đó chạy nhất quán.
- Danh sách quyền AWS nên có hoạt động như một checklist sẵn sàng về IAM cho người làm workshop.
- Danh sách nội dung định nghĩa đúng thứ tự phân rã kiến trúc: edge trước, runtime sau, data tiếp theo, rồi tới operations và observability.

Phụ thuộc:

- Có AWS account và region hoạt động bình thường.
- Có naming convention đủ rõ cho Route 53, CloudFront, EC2, RDS và SES.
- Có sự thống nhất rằng workshop này mô tả hành vi của môi trường đã deploy, không phải local setup.

### 4.1 Tổng quan workshop và bản đồ dịch vụ

![AWS Workshop Architecture Overview](/images/4-Workshop/ev-achi.png)

Phần này làm gì:

- Định nghĩa bản đồ dịch vụ đầu cuối của EV Charging Station Management System.
- Tách bạch các thành phần public với runtime và data tier private.
- Giới thiệu vòng đời request chính trước khi người đọc đi vào từng nhóm dịch vụ cụ thể.

Vì sao phần này tồn tại:

- Người đọc cần một mental model trước khi đi debug từng lớp.
- Nó đặt toàn bộ workshop vào đúng ngữ cảnh của một hệ thống nhiều tầng thay vì một danh sách rời rạc các sản phẩm AWS.
- Nó xác nhận quan điểm kiến trúc rằng backend đang là ALB cộng với EC2 chứ không phải API Gateway cộng với Lambda.

Hành vi đầu vào và đầu ra:

| Đầu vào hoặc kích hoạt | Xử lý trong phần này | Đầu ra hoặc tác dụng |
| --- | --- | --- |
| Người mới cần định hướng | Kiến trúc nhiều lớp được tóm tắt | Người đọc đặt được từng phần sau vào đúng ngữ cảnh |
| Nhóm cần ánh xạ nhu cầu sang dịch vụ | Bảng bản đồ dịch vụ AWS được cung cấp | Mỗi dịch vụ được gắn với một trách nhiệm kiến trúc cụ thể |
| Nhóm cần kiểm tra nhanh môi trường | Bộ lệnh CLI nền tảng được đưa ra | Operator xác minh account, region và tài nguyên chính trước khi đi sâu hơn |

Giải thích ở mức cao về các lệnh trong workshop:

- `aws sts get-caller-identity` xác nhận operator đang đứng ở đúng account.
- `aws route53 list-hosted-zones`, `aws cloudfront list-distributions`, `aws ec2 describe-instances` và `aws rds describe-db-instances` là bộ kiểm tra tồn tại cơ bản cho những trụ cột chính của kiến trúc.
- Các lệnh này đều là read-only, nên mục `4.1` đóng vai trò như một bước pre-flight an toàn.

Phụ thuộc:

- Hosted zone trong Route 53 tồn tại.
- CloudFront distribution tồn tại và gắn với domain mong muốn.
- EC2 instance và RDS instance tồn tại trong account và region mục tiêu.
- Naming convention trong workshop khớp với môi trường thật.

### 4.2 Phân phối biên và yêu cầu dữ liệu bản đồ

#### Amazon Route 53

![Amazon Route 53 DNS](/images/4-Workshop/route53.jpg)

#### Amazon CloudFront

![Amazon CloudFront Distribution](/images/4-Workshop/cloudfront.jpg)

#### Amazon S3

![Amazon S3 Bucket](/images/4-Workshop/s3-bucket.jpg)

#### Amazon Location Service

![Amazon Location Service](/images/4-Workshop/location-service.png)

Phần này làm gì:

- Xử lý đường đi public đầu tiên cho trình duyệt người dùng.
- Phân phối frontend asset với cache toàn cầu và origin S3 private.
- Bảo vệ đường edge bằng WAF trước khi request chạm origin.
- Tách luồng map và place search ra khỏi luồng phân phối static asset thông thường.

Vì sao phần này tồn tại:

- Frontend delivery có đặc tính scale và độ trễ rất khác với backend API execution.
- Static asset nên được cache toàn cầu và phục vụ với chi phí thấp, còn backend chỉ nên xử lý traffic động.
- Geospatial request dựa vào một dịch vụ chuyên biệt của AWS nên không nên bị ép đi chung với đường static file.

Luồng tương tác dịch vụ:

1. Trình duyệt phân giải `${APP_DOMAIN}` qua Route 53.
2. Route 53 trỏ request tới CloudFront.
3. CloudFront đánh giá WAF rule trước khi phục vụ hoặc chuyển tiếp request.
4. Nếu cache hit thì CloudFront trả object ngay.
5. Nếu cache miss thì CloudFront đọc object từ S3 qua Origin Access Control.
6. Sau khi SPA được tải xong, frontend gọi Amazon Location Service để render bản đồ hoặc tìm địa điểm.
7. Những request API tiếp theo rời khỏi module edge và đi sang tầng ứng dụng dựa trên ALB.

Hành vi đầu vào và đầu ra:

| Đầu vào | Đường xử lý | Đầu ra |
| --- | --- | --- |
| DNS lookup cho `${APP_DOMAIN}` | Route 53 phân giải alias hoặc CNAME | CloudFront hostname được trả về cho client |
| HTTPS request lấy `index.html`, JS, CSS hoặc media | CloudFront phục vụ từ cache hoặc lấy từ S3 | Trình duyệt nhận asset tĩnh với CDN acceleration |
| Mẫu request HTTP đáng ngờ | WAF kiểm tra trước khi cho vào origin | Request được cho qua, chặn, đếm hoặc challenge tùy policy |
| Place search hoặc map request | Frontend gọi Amazon Location Service | Trình duyệt nhận kết quả geospatial hoặc nội dung bản đồ |

Giải thích ở mức cao về các lệnh và cấu hình mẫu:

- Lệnh kiểm tra hosted zone xác nhận quyền sở hữu DNS public đang tồn tại.
- Mẫu `route53-app-record.json` dạy cách quản lý DNS theo kiểu idempotent bằng `UPSERT`, rất phù hợp cho thao tác hạ tầng.
- `aws s3 sync ./dist s3://${STATIC_BUCKET} --delete` mô hình hóa một pipeline deploy frontend ở dạng đơn giản.
- Lệnh invalidation của CloudFront giải thích vì sao thay file trong S3 thôi là chưa đủ khi CDN cache còn tồn tại.
- Lệnh tìm kiếm với Location Service chứng minh luồng bản đồ tách biệt với cả S3 lẫn ALB.
- Bucket policy của S3 là artifact bảo mật quan trọng nhất của module này vì nó ép người dùng đi qua CloudFront thay vì đọc bucket trực tiếp.

Phụ thuộc:

- Hosted zone và record public của ứng dụng.
- CloudFront distribution có certificate hợp lệ và đã gắn WAF.
- S3 bucket private với quyền đọc dành riêng cho OAC.
- Place index và cấu hình map trong Amazon Location Service.
- Kỷ luật deploy luôn invalidate asset sau mỗi lần phát hành frontend.

### 4.3 Điều phối ứng dụng riêng tư và tầng tính toán

#### Amazon VPC

![Amazon VPC](/images/4-Workshop/vpc-overview.png)

#### Internet Gateway

![Internet Gateway](/images/4-Workshop/internet-gateway.png)

#### NAT Gateway

![NAT Gateway](/images/4-Workshop/nat-gateway.png)

#### Application Load Balancer

![Application Load Balancer](/images/4-Workshop/alb.png)

#### Amazon EC2

![Amazon EC2 Instances](/images/4-Workshop/ec2-instances.png)

#### EC2 Auto Scaling

![Auto Scaling Group](/images/4-Workshop/auto-scaling-group.png)

Phần này làm gì:

- Chuyển dynamic traffic của người dùng từ edge vào compute private của ứng dụng.
- Định nghĩa cách public subnet và private subnet tương tác bên trong VPC.
- Đảm bảo chỉ ALB là internet-facing còn EC2 runtime vẫn nằm trong private subnet.
- Dùng Auto Scaling và health check để backend vừa scale ngang được vừa có khả năng tự phục hồi.

Vì sao phần này tồn tại:

- API động cần routing có kiểm soát, đánh giá health và khả năng scale ngang.
- Private subnet giúp giảm bề mặt phơi lộ của application server.
- Backend vẫn cần outbound internet cho package, endpoint public của AWS hoặc tích hợp bên thứ ba, vì vậy NAT là thành phần cần thiết.

Luồng tương tác dịch vụ:

1. Frontend gửi HTTPS API request tới entry point của backend.
2. ALB nhận request trên listener public.
3. Listener rule và trạng thái target-group quyết định EC2 nào sẽ xử lý.
4. Một EC2 khỏe mạnh trong private subnet thực thi business logic.
5. Nếu ứng dụng cần đi ra ngoài, traffic đi qua NAT Gateway rồi ra Internet Gateway.
6. Ứng dụng trả response cho ALB và ALB trả tiếp về client.
7. Auto Scaling thay instance lỗi hoặc tăng capacity khi load tăng.

Hành vi đầu vào và đầu ra:

| Đầu vào | Đường xử lý | Đầu ra |
| --- | --- | --- |
| HTTPS API request từ frontend | Listener của ALB route tới target healthy | Response nghiệp vụ được trả về cho caller |
| Health-check probe từ ALB | Endpoint health của EC2 phản hồi | Target tiếp tục healthy hoặc bị loại khỏi rotation |
| Outbound call từ EC2 private | Route table private đẩy egress qua NAT Gateway | Instance truy cập được endpoint bên ngoài mà không lộ public IP |
| Sự kiện tăng tải hoặc lỗi instance | Auto Scaling tạo mới hoặc thay thế instance | Desired count và độ phủ nhiều AZ được duy trì |

Giải thích ở mức cao về các lệnh và cấu hình mẫu:

- `describe-load-balancers` xác nhận ALB tồn tại, có internet-facing và dùng đúng subnet.
- `describe-target-health` là lệnh kiểm tra runtime quan trọng nhất của module này vì nó cho biết request có thực sự đi được tới backend healthy hay không.
- `describe-auto-scaling-groups` cho thấy desired capacity, subnet placement và lifecycle state của nhóm backend.
- `describe-instances` giúp soi chi tiết từng node khi cần so sánh một instance với baseline của cả nhóm.
- Bảng security group và route table chính là logic kiến trúc cốt lõi của phần này vì nó quyết định ai được nói chuyện với ai.
- Bộ health-check setting khuyến nghị dạy rằng liveness phải nhẹ và không nên chết chỉ vì database đang bị stress.

Phụ thuộc:

- VPC có ít nhất một public subnet và ít nhất hai private subnet để bố trí ứng dụng bền vững hơn.
- Có listener, target group và endpoint health check trên ALB.
- Có launch template, AMI, instance profile và security group cho EC2.
- Có NAT Gateway và route table đúng cho egress của private subnet.
- Ứng dụng hỗ trợ scale ngang theo kiểu stateless hoặc rất ít phụ thuộc session.

### 4.4 Độ bền dữ liệu quan hệ và cô lập mạng

#### Amazon RDS

![Amazon RDS](/images/4-Workshop/rds-overview.png)

#### Primary Database Instance

![RDS Primary Instance](/images/4-Workshop/rds-primary.png)

Phần này làm gì:

- Mô tả hệ thống lưu trữ quan hệ trung tâm của ứng dụng.
- Xác lập rằng database tier là private và chỉ được truy cập từ các tài nguyên nội bộ đã được phê duyệt.
- Giới thiệu mô hình primary và standby để tăng độ bền khi database hoặc hạ tầng bên dưới gặp sự cố.

Vì sao phần này tồn tại:

- Các nghiệp vụ như tài khoản người dùng, booking, charging session, payment và dữ liệu vận hành đều cần tính nhất quán giao dịch.
- Lưu trữ quan hệ phù hợp hơn object storage hoặc key-value store cho workload đang được workshop mô tả.
- Data tier là ranh giới lưu trữ có giá trị cao nhất nên bắt buộc phải được cô lập.

Diễn giải kiến trúc:

- Chính chương workshop mô tả RDS theo mô hình primary và standby, điều này hàm ý tư thế Multi-AZ hoặc một chế độ HA tương đương.
- Mô hình đó bền vững hơn so với proposal trước đây, nơi từng nhắc tới SQL Server Single-AZ.
- Trong tài liệu này, phần `4-Workshop` được xem là nguồn mới hơn và có độ trưởng thành vận hành cao hơn.

Luồng tương tác dịch vụ:

1. Ứng dụng trên EC2 lấy thông tin kết nối database từ Secrets Manager.
2. Ứng dụng mở kết nối mạng private tới endpoint của RDS.
3. Database instance chính xử lý giao dịch đọc và ghi.
4. RDS đồng bộ trạng thái sang vùng standby theo chế độ khả dụng đã chọn.
5. Nếu primary gặp lỗi, RDS thực hiện failover và cố gắng giữ ổn định endpoint ở mức tối đa.
6. Ứng dụng reconnect bằng endpoint quản lý của RDS thay vì hostname gắn cứng theo từng máy.

Hành vi đầu vào và đầu ra:

| Đầu vào | Đường xử lý | Đầu ra |
| --- | --- | --- |
| SQL read hoặc write từ EC2 | Request đi tới primary RDS endpoint qua mạng private | Ứng dụng nhận kết quả truy vấn hoặc xác nhận transaction |
| Thay đổi trạng thái dữ liệu | RDS đồng bộ sang hạ tầng standby | Tăng độ bền và rút ngắn thời gian phục hồi sau lỗi |
| Lỗi primary hoặc sự kiện bảo trì | Đường failover được quản lý kích hoạt | Tính liên tục dịch vụ thông qua chuyển vai trò sang standby |
| Request trái phép từ internet | Không có đường public nào tới database | Database vẫn không thể truy cập từ internet công khai |

Giải thích ở mức cao về các lệnh và cấu hình mẫu:

- `describe-db-instances` là lệnh soi cấu trúc chính của module này.
- Ví dụ `--query` trả về trạng thái, cờ `MultiAZ`, endpoint và port đặc biệt có giá trị vì nó kiểm chứng trực tiếp việc môi trường thật có khớp với tuyên bố về độ bền hay không.
- `reboot-db-instance --force-failover` là một bài test chịu lỗi có kiểm soát, không phải thao tác quản trị thường ngày.
- Mẫu JSON secret mô hình hóa việc tách biệt application code với thông tin kết nối.
- Bảng cấu hình nhấn mạnh rằng ứng dụng phải phụ thuộc vào DNS endpoint của RDS chứ không phải IP của một instance cụ thể.

Phụ thuộc:

- Có DB subnet group private trải trên nhiều subnet.
- Chỉ mở security group từ tầng ứng dụng.
- Backup retention, parameter group và lịch bảo trì phải phù hợp với mục tiêu production.
- Ứng dụng có connection pooling và reconnect logic đủ tốt để chịu được các khoảng gián đoạn failover ngắn.

### 4.5 Truy cập vận hành và quản trị bí mật

#### AWS Systems Manager

![AWS Systems Manager Session Manager](/images/4-Workshop/ssm-session-manager.png)

#### IAM Instance Profile

![IAM Instance Profile](/images/4-Workshop/iam-instance-profile.png)

#### Amazon EC2 Runtime Identity

![EC2 Instance IAM Role](/images/4-Workshop/ec2-iam-role.png)

Phần này làm gì:

- Xác định cách con người và máy có được quyền truy cập có kiểm soát vào một môi trường private.
- Thay thế việc phát tán SSH key hoặc bastion public bằng Systems Manager và IAM.
- Đưa cấu hình nhạy cảm ra khỏi file ứng dụng và đặt vào hệ secret được quản lý.

Vì sao phần này tồn tại:

- Workload production không nên phụ thuộc vào hardcoded credential hoặc đường quản trị công khai.
- Danh tính máy phải là tạm thời, có phạm vi hẹp và tự xoay vòng.
- Việc lưu secret và mã hóa nên được giao cho dịch vụ managed của AWS thay vì tự làm lại ở application layer.

Luồng tương tác dịch vụ:

1. Quản trị viên khởi tạo truy cập qua Systems Manager.
2. EC2 mục tiêu xác thực bằng instance profile được gắn kèm.
3. Ứng dụng hoặc operator đọc secret từ Secrets Manager theo tên.
4. KMS bảo vệ dữ liệu secret bên dưới và chỉ cho phép principal được quyền giải mã.
5. Cùng IAM role đó cũng có thể ghi log hoặc gửi email khi cần.

Hành vi đầu vào và đầu ra:

| Đầu vào | Đường xử lý | Đầu ra |
| --- | --- | --- |
| Admin cần shell access vào EC2 private | Session Manager tạo phiên truy cập | Có remote access an toàn mà không cần SSH public |
| EC2 khởi động hoặc gọi AWS API trong runtime | Instance profile cấp temporary credential | Ứng dụng gọi AWS API an toàn |
| Ứng dụng cần credential hoặc token | Secrets Manager trả secret sau khi kiểm tra quyền | Runtime nạp cấu hình mà không nhúng secret vào code |
| Quá trình đọc secret chạm dữ liệu mã hóa | KMS áp dụng quyền ở mức key | Chỉ role hợp lệ mới giải mã được secret |

Giải thích ở mức cao về các lệnh và cấu hình mẫu:

- `aws ssm start-session` chứng minh đường vận hành không cần bastion host.
- Lệnh kiểm tra instance profile cho EC2 xác nhận danh tính runtime kỳ vọng có thực sự được gắn hay không.
- `aws secretsmanager get-secret-value` là cách trực diện nhất để xác minh secret name, policy truy cập và region.
- `aws kms list-aliases` chưa phải toàn bộ câu chuyện audit của KMS, nhưng là cách thực tế để kiểm tra cấu trúc đặt tên khóa.
- Policy IAM tối thiểu minh họa triết lý cấp quyền cho máy trong workshop: đọc secret, giải mã, ghi log và gửi email, hoàn toàn không cần lưu static AWS key trên host.

Phụ thuộc:

- EC2 phải được đăng ký và ở trạng thái healthy trong Systems Manager.
- Instance profile cần trust policy đúng và quyền theo nguyên tắc least privilege.
- Cách đặt tên secret, cách tách theo môi trường và ownership của KMS key phải nhất quán.
- Nếu môi trường bị khóa chặt, nên cân nhắc VPC endpoint cho Secrets Manager, SSM và CloudWatch thay vì phụ thuộc hoàn toàn vào NAT.

### 4.6 Giám sát, kiểm toán và gửi email

#### Amazon CloudWatch

![Amazon CloudWatch Logs and Metrics](/images/4-Workshop/cloudwatch.png)

#### AWS CloudTrail

![AWS CloudTrail Events](/images/4-Workshop/cloudtrail.png)

#### Amazon SES

![Amazon SES](/images/4-Workshop/ses.png)

#### Route 53 Email DNS Records

![Route 53 SES DNS Records](/images/4-Workshop/route53-ses.png)

Phần này làm gì:

- Xác định cách nền tảng được quan sát trong runtime.
- Ghi lại hoạt động control-plane của AWS cho mục tiêu governance và điều tra bảo mật.
- Gửi mail do ứng dụng phát sinh qua SES với xác thực domain dựa trên Route 53.

Vì sao phần này tồn tại:

- Một hệ thống production chưa hoàn chỉnh nếu không thể quan sát, kiểm toán và tin cậy vào đường gửi thông báo.
- Email thường là tính năng nghiệp vụ quan trọng cho account event, cập nhật booking hoặc cảnh báo vận hành.
- Audit đặc biệt quan trọng vì workshop phụ thuộc mạnh vào đường quản trị native của AWS như SSM, IAM và các API thay đổi hạ tầng.

Luồng tương tác dịch vụ:

1. Ứng dụng ghi log và metric lên CloudWatch.
2. Operator xem log, alarm và metric để chẩn đoán hành vi production.
3. Các dịch vụ AWS ghi control-plane activity vào CloudTrail.
4. Workload trên EC2 gửi mail qua SES bằng quyền IAM của chính nó.
5. SES phụ thuộc vào record trong Route 53 để verify identity, DKIM, SPF và hỗ trợ mail routing.

Hành vi đầu vào và đầu ra:

| Đầu vào | Đường xử lý | Đầu ra |
| --- | --- | --- |
| Sự kiện log hoặc metric từ ứng dụng | CloudWatch lưu telemetry | Operator có thể search, cảnh báo và dựng dashboard |
| Hành động AWS API mang tính quản trị | CloudTrail ghi nhận event | Nhóm có thể audit ai đổi gì và vào thời điểm nào |
| Yêu cầu gửi email outbound | SES xử lý nếu identity đã verify | Người nhận nhận được thông báo và hệ thống không cần tự quản lý SMTP |
| Nhu cầu xác thực domain cho SES | Route 53 công bố TXT, CNAME và MX record | Identity của SES được xác minh và khả năng deliverability tốt hơn |

Giải thích ở mức cao về các lệnh và cấu hình mẫu:

- `aws logs tail` là lệnh hỗ trợ debug trực tiếp nhanh nhất trong cả chương vì nó rút ngắn vòng phản hồi khi có sự cố.
- `aws cloudtrail lookup-events` nối hành động vận hành với bằng chứng audit, đặc biệt hữu ích khi thay đổi xảy ra bên ngoài application code.
- `aws sesv2 get-email-identity` kiểm tra độ sẵn sàng của đường mail trước khi ứng dụng bắt đầu gửi.
- Payload `ses-test-email.json` là một hợp đồng mail tối thiểu để chứng minh role, identity và region đang khớp nhau.
- `put-metric-alarm` cho thấy observability không chỉ là đọc log mà còn là biến tín hiệu thành cảnh báo mà operator nhìn thấy được.
- Mẫu JSON Route 53 cho SES là ví dụ rõ nhất cho thấy gửi mail không chỉ là chuyện của code ứng dụng mà còn là bài toán hạ tầng.

Phụ thuộc:

- Có log group, retention setting và permission cho CloudWatch.
- Có CloudTrail được bật với chiến lược lưu giữ dữ liệu audit phù hợp.
- Có SES identity đã verify và DNS record đúng trong Route 53.
- Ứng dụng hoặc agent thực sự phát ra log, metric và nội dung message có giá trị.

## Vòng đời request và luồng tương tác dịch vụ

### Vòng đời 1: Phân phối frontend tĩnh

1. Người dùng nhập domain ứng dụng trên trình duyệt.
2. Route 53 phân giải tên về CloudFront.
3. CloudFront kết thúc HTTPS và áp dụng WAF control.
4. CloudFront trả asset đã cache hoặc đọc từ S3 thông qua OAC.
5. Trình duyệt khởi động frontend application.

Vì sao vòng đời này quan trọng:

- Nó giảm độ trễ cho người dùng phân bố rộng.
- Nó giữ cho S3 bucket không bị public.
- Nó giảm lượng traffic động không cần thiết vào backend.

### Vòng đời 2: Trải nghiệm geospatial

1. SPA yêu cầu map tile, place lookup hoặc search suggestion.
2. Request được gửi tới Amazon Location Service thay vì đi qua tầng EC2 của ứng dụng.
3. Frontend render ngữ cảnh bản đồ hoặc kết quả tìm kiếm cho người dùng.

Vì sao vòng đời này quan trọng:

- Nó giữ workload geospatial ở đúng dịch vụ chuyên biệt và tách khỏi API giao dịch cốt lõi.
- Nó tránh biến backend thành proxy không cần thiết cho dữ liệu bản đồ.

### Vòng đời 3: Dynamic API request

1. Frontend gọi backend endpoint được công bố qua ALB.
2. ALB chọn một target EC2 private đang healthy.
3. Ứng dụng trên EC2 nạp cấu hình hoặc secret nếu cần.
4. Ứng dụng thực thi business logic và làm việc với RDS.
5. Ứng dụng ghi log rồi trả response ngược qua ALB.

Vì sao vòng đời này quan trọng:

- Đây là đường đi nghiệp vụ cốt lõi của hệ thống.
- Nó tập trung vùng public exposure tại ALB thay vì tại instance layer.

### Vòng đời 4: Giao dịch gắn với database

1. EC2 lấy secret database hiện tại.
2. Ứng dụng kết nối tới endpoint của RDS qua private network.
3. Primary instance xử lý transaction.
4. RDS duy trì tính liên tục của standby.
5. Nếu có failover, ứng dụng reconnect bằng managed endpoint.

Vì sao vòng đời này quan trọng:

- Nó giữ được tính toàn vẹn giao dịch.
- Nó ẩn thay đổi host database phía sau một endpoint quản lý ổn định.

### Vòng đời 5: Truy cập vận hành và chẩn đoán

1. Quản trị viên mở một phiên Systems Manager tới EC2 private.
2. Instance dùng IAM role của mình để xác thực với các dịch vụ AWS.
3. Operator kiểm tra trạng thái ứng dụng, log cục bộ hoặc cấu hình runtime.
4. Các hoạt động liên quan sau đó có thể được nhìn thấy trong CloudTrail và CloudWatch.

Vì sao vòng đời này quan trọng:

- Nó cho phép quản trị mà không cần SSH public.
- Nó giữ cho thao tác vận hành có thể audit được.

### Vòng đời 6: Hoàn tất email và audit

1. Một sự kiện nghiệp vụ trong ứng dụng kích hoạt việc gửi mail.
2. Workload trên EC2 gọi SES bằng IAM role của chính instance.
3. SES chấp nhận message khi sender identity và DNS record hợp lệ.
4. Control-plane action vẫn được nhìn thấy trong CloudTrail.
5. Tín hiệu thành công hoặc lỗi phía ứng dụng được ghi vào CloudWatch.

Vì sao vòng đời này quan trọng:

- Nó khép lại vòng lặp giữa hành động nghiệp vụ, giao tiếp với người dùng và khả năng quan sát của operator.

## Các cân nhắc khi triển khai

| Hạng mục triển khai | Tư thế hiện tại trong workshop | Cách xử lý nên có ở production |
| --- | --- | --- |
| Frontend | Thao tác thủ công hoặc CLI bằng `aws s3 sync` cộng với CloudFront invalidation | Build asset có version, publish lên S3, chỉ invalidate path thật sự cần và giữ version cũ để rollback |
| Backend compute | EC2 nằm sau ALB và Auto Scaling | Dùng launch template với artifact bất biến, triển khai rolling hoặc instance refresh và chỉ xóa node cũ sau khi health check xanh |
| Database change | RDS là hệ thống lưu trữ trung tâm nhưng workshop chưa cho thấy tool migration | Chạy migration trong một release stage có kiểm soát qua SSM hoặc CI/CD, đi kèm backup và kế hoạch rollback |
| Secret và cấu hình | Secret được tập trung trong Secrets Manager | Tách secret theo môi trường, xoay vòng định kỳ và không trộn credential với cấu hình tĩnh |
| Certificate và DNS | HTTPS được ngầm hiểu và Route 53 đã nằm trong phạm vi | Quản lý certificate cho CloudFront và ALB bằng ACM, đồng thời tự động hóa DNS change cẩn thận để giảm rủi ro cutover |
| Quy trình vận hành | Chương workshop dùng CLI inspection rất nhiều | Chuyển các bước lặp lại thành runbook, kiểm tra bằng Infrastructure as Code và dashboard |

Trình tự triển khai được khuyến nghị:

1. Build và test frontend artifact.
2. Publish frontend lên S3 rồi invalidate các path cần thiết trên CloudFront.
3. Build backend artifact và cập nhật launch template hoặc machine image cho EC2.
4. Roll Auto Scaling Group từng phần trong khi theo dõi target health của ALB và log trên CloudWatch.
5. Chỉ chạy database migration sau khi backup posture và bước rollback đã được xác nhận.
6. Xác minh SES identity, alarm và đường truy cập vận hành trước khi chốt release là healthy.

## Khả năng mở rộng và chịu lỗi

| Mối quan tâm | Tư thế hiện tại trong workshop | Lợi ích | Khoảng trống hoặc rủi ro hiện tại | Cách hardening khuyến nghị |
| --- | --- | --- | --- | --- |
| Scale nội dung tĩnh | CloudFront cộng với S3 | Edge caching hấp thụ lượng lớn traffic đọc toàn cầu rất hiệu quả | Sau mỗi release vẫn phải duy trì kỷ luật invalidation | Dùng asset versioning và cache policy được tinh chỉnh |
| Scale ứng dụng | ALB cộng với Auto Scaling trên nhiều AZ | Có scale ngang và thay instance lỗi | Nếu ứng dụng nặng session thì hiệu quả scale sẽ giảm | Giữ node stateless và chỉnh scaling policy phù hợp |
| Độ bền database | RDS với tư thế primary và standby | Managed failover tăng độ bền và khả năng phục hồi | Cần xác nhận Multi-AZ thật trong môi trường triển khai | Kiểm tra định kỳ Multi-AZ, backup restore và failover drill |
| Egress private | NAT Gateway cho outbound traffic | Instance vẫn private nhưng đi được ra ngoài | Một NAT Gateway duy nhất là trade-off về cả chi phí lẫn độ sẵn sàng | Dùng NAT Gateway theo từng AZ hoặc thêm VPC endpoint để giảm lệ thuộc vào NAT |
| Truy cập secret | Secrets Manager và KMS | Tập trung hóa secret và mã hóa | IAM quá rộng sẽ làm yếu mô hình bảo mật | Thu hẹp resource scope và bật rotation |
| Truy cập operator | Systems Manager | Không cần bastion host hay SSH key rải rác | SSM phụ thuộc vào agent và IAM đúng | Bật session logging và kiểm tra quy trình break-glass |
| Observability | CloudWatch và CloudTrail | Log, metric và audit event được tập trung | Workshop chưa thể hiện alarm coverage và retention policy | Bổ sung dashboard, alarm routing, retention class và metric filter |
| Gửi email | SES cộng với DNS verification | Email giao dịch được đưa ra dịch vụ managed | Chưa thấy bounce handling và complaint workflow | Thêm feedback processing và theo dõi reputation |
| Cô lập workload bất đồng bộ | Chưa có | Kiến trúc hiện tại đơn giản hơn | Email, retry hoặc side effect có thể vẫn còn gắn chặt với độ trễ của request | Bổ sung EventBridge, SQS hoặc SNS khi cần xử lý nền |

## Cơ hội cải thiện và đối chiếu với AWS Well-Architected

| Trụ cột AWS Well-Architected | Điểm mạnh hiện có trong workshop | Bước tiếp theo nên làm |
| --- | --- | --- |
| Operational Excellence | Chương này đã khuyến khích CLI verification, dùng biến môi trường và phân rõ trách nhiệm dịch vụ | Chuyển các bước kiểm tra thành Infrastructure as Code, health validation tự động và runbook vận hành tái sử dụng |
| Security | EC2 và RDS private, WAF, SSM, IAM instance profile, Secrets Manager và KMS tạo nền tảng khá chắc | Siết IAM chặt hơn, bật session logging, xoay vòng secret, thêm VPC endpoint cho dịch vụ nhạy cảm và quản lý certificate rõ ràng |
| Reliability | ALB health check, Auto Scaling và tư thế RDS có standby giúp tăng sức chịu lỗi | Xác nhận chế độ HA thật của database, test failover thường xuyên, loại bỏ điểm yếu NAT một AZ và cân nhắc tách side effect không quan trọng sang luồng bất đồng bộ |
| Performance Efficiency | CloudFront caching và EC2 scale-out hỗ trợ khá tốt cho workload web chính | Tối ưu cache policy, connection pooling cho database và cân nhắc lớp read optimization nếu lưu lượng tăng mạnh |
| Cost Optimization | S3, CloudFront và các managed service giảm bớt gánh nặng vận hành | Rà soát chi phí NAT, right-size EC2 và RDS, dùng lifecycle policy và thay thế lưu lượng public endpoint không cần thiết bằng VPC endpoint nếu phù hợp |
| Sustainability | Các dịch vụ co giãn và managed tránh được nhiều hạ tầng tự vận hành luôn bật | Lên lịch tắt môi trường non-production, tối ưu cache hit ratio và định kỳ right-size compute chạy lâu dài |

### Hướng tiến hóa tự nhiên

- Chỉ đưa API Gateway vào khi nền tảng thực sự cần các tính năng quản trị API public mà ALB không làm tốt một cách tự nhiên.
- Có thể thêm Lambda cho các automation nhỏ như hậu xử lý nhẹ, xử lý sự kiện SES hoặc hỗ trợ secret rotation.
- Có thể thêm Step Functions khi luồng booking, charging, payment hoặc compliance trở nên nhiều bước và kéo dài.
- Có thể thêm EventBridge, SQS hoặc SNS khi notification, retry hoặc background processing không nên còn gắn với độ trễ của request chính.
- Có thể thêm read replica, Aurora hoặc kho báo cáo riêng khi throughput quan hệ hoặc nhu cầu tách workload thực sự đòi hỏi.

## Checklist sẵn sàng vận hành

- Xác minh đường public đang khỏe: Route 53 phân giải đúng domain, CloudFront dùng đúng certificate, WAF đã gắn và S3 vẫn được giữ private phía sau OAC.
- Xác minh đường ứng dụng đang khỏe: listener và target group của ALB ở trạng thái tốt, Auto Scaling trải trên các Availability Zone mong muốn và EC2 private đi được tới các dependency cần thiết.
- Xác minh đường dữ liệu đang khỏe: endpoint RDS là private, backup đã bật, tư thế failover được hiểu rõ và ứng dụng đọc credential từ Secrets Manager thay vì file cục bộ.
- Xác minh control plane đang khỏe: truy cập SSM hoạt động mà không cần SSH, IAM role tuân thủ least privilege, KMS policy được giới hạn đúng phạm vi và CloudTrail ghi nhận đầy đủ thao tác quản trị.
- Xác minh observability đang khỏe: CloudWatch log group tồn tại với retention phù hợp, alarm quan trọng có owner, SES identity đã verify và Route 53 có đầy đủ DNS record hỗ trợ email.

## Ghi chú / Giả định

- Tài liệu này bám trên source của workshop dưới `content/4-Workshop/`, không phải source code của backend application.
- Workshop mô tả RDS với tư thế bền vững hơn proposal trước đó. Tài liệu này lấy workshop làm mục tiêu kiến trúc hiện tại.
- Triển khai đang hiển thị có chủ đích AWS-native và chủ yếu là đồng bộ.
- API Gateway, Lambda, ECS, EKS, DynamoDB, Step Functions, EventBridge, SQS, SNS và Cognito chưa phải một phần của kiến trúc hiện tại đã được xác nhận, nhưng nhiều dịch vụ trong số đó là hướng mở rộng hợp lý khi hệ thống lớn hơn.
- Security group, ACM certificate, DB subnet group, launch template và các tích hợp ở mức agent được coi là thành phần suy ra bắt buộc vì các luồng AWS được mô tả sẽ không thể vận hành đúng nếu thiếu chúng.
