---
title: "Proposal"
date: 2026-03-24
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Hệ thống quản lý trạm sạc xe điện

## Nền tảng backend thông minh cho vận hành trạm sạc và thanh toán số

### 1. Tóm tắt điều hành

Hệ thống quản lý trạm sạc xe điện là một nền tảng backend tập trung được xây dựng nhằm hỗ trợ toàn bộ vòng đời vận hành của dịch vụ sạc xe điện. Hệ thống bao phủ các nghiệp vụ cốt lõi như tìm kiếm trạm sạc, quản lý khung giờ, tạo và xác nhận đặt chỗ, theo dõi phiên sạc, tạo hóa đơn, xử lý thanh toán và quản lý tuân thủ của tài xế trong một nền tảng thống nhất.

Được phát triển trên Spring Boot và tích hợp với hạ tầng cloud, hệ thống mang lại sự cân bằng giữa độ ổn định vận hành, tính bảo mật và khả năng mở rộng về lâu dài. Nền tảng hỗ trợ phân quyền theo vai trò cho tài xế, nhân viên vận hành và quản trị viên; tự động hóa việc quản lý slot và vòng đời phiên sạc; đồng thời tích hợp VNPay cho thanh toán trực tuyến, Amazon S3 và CloudFront cho lưu trữ và phân phối nội dung frontend, Amazon RDS for SQL Server cho cơ sở dữ liệu quản lý tập trung, AWS Map cho tích hợp bản đồ, cùng dịch vụ email SMTP cho các thông báo giao dịch và vận hành.

Giải pháp này đặc biệt phù hợp với các đơn vị khai thác trạm sạc EV đang muốn số hóa quy trình vận hành, nâng cao hiệu suất sử dụng trạm, giảm các thao tác thủ công, chuẩn hóa quy trình dịch vụ và chuẩn bị nền tảng kỹ thuật để mở rộng trong tương lai.

### 2. Bài toán đặt ra

#### Vấn đề hiện tại là gì?

Khi thị trường xe điện phát triển nhanh, các đơn vị vận hành trạm sạc phải đối mặt với áp lực ngày càng lớn trong việc quản lý công suất trạm, lịch đặt chỗ, phiên sạc, hóa đơn, thanh toán và dịch vụ hỗ trợ người dùng. Nếu các hoạt động này vẫn được thực hiện thủ công hoặc phân tán trên nhiều hệ thống riêng lẻ, doanh nghiệp sẽ gặp phải nhiều vấn đề đáng kể:

- Khó theo dõi tình trạng slot trống theo thời gian thực
- Quy trình đặt chỗ, xác nhận và tiếp nhận người dùng còn rời rạc, kém hiệu quả
- Chậm trễ trong việc tạo hóa đơn và đối soát thanh toán
- Khó kiểm soát các chính sách vận hành như quá giờ sạc hoặc vi phạm sử dụng
- Thiếu công cụ giữ chân khách hàng như chương trình điểm thưởng
- Gặp khó khăn trong việc quản lý tập trung nhiều nhóm người dùng như tài xế, nhân viên và quản trị viên
- Thiếu kiến trúc số thống nhất kết nối frontend, backend, bản đồ và cơ sở dữ liệu trong cùng một hệ thống

Nếu không có một hệ thống backend tập trung, đơn vị vận hành có nguy cơ khai thác hạ tầng chưa hiệu quả, chất lượng dịch vụ thiếu nhất quán, khó mở rộng quy mô và suy giảm trải nghiệm người dùng.

#### Giải pháp đề xuất

Giải pháp được đề xuất là một hệ thống backend quản lý vận hành trạm sạc EV theo mô hình tập trung, có khả năng điều phối toàn bộ quy trình từ đầu đến cuối. Hệ thống cung cấp đầy đủ các năng lực cần thiết để vận hành dịch vụ một cách đồng bộ, bao gồm:

- Xác thực người dùng an toàn và phân quyền theo vai trò
- Quy trình đặt chỗ và giữ slot rõ ràng, có kiểm soát
- Tự động quản lý vòng đời phiên sạc
- Tạo hóa đơn và tính phí theo nghiệp vụ
- Tích hợp thanh toán trực tuyến qua VNPay
- Theo dõi vi phạm tài xế và thực thi chính sách tuân thủ
- Quản lý điểm thưởng nhằm tăng mức độ quay lại sử dụng
- Lưu trữ tệp và tài nguyên tĩnh trên Amazon S3
- Phân phối nội dung thông qua CloudFront
- Tích hợp bản đồ bằng AWS Map
- Tự động hóa sinh slot và xử lý booking quá hạn bằng các tác vụ theo lịch

Thông qua việc tập trung các chức năng này vào một nền tảng thống nhất, doanh nghiệp có thể chuẩn hóa vận hành, cải thiện chất lượng dịch vụ, tăng năng lực kiểm soát và xây dựng nền tảng số sẵn sàng cho tăng trưởng.

#### Lợi ích và hiệu quả đầu tư

Giải pháp mang lại cả giá trị vận hành lẫn giá trị kinh doanh trong ngắn hạn và dài hạn:

- **Tăng hiệu suất sử dụng trạm** nhờ quản lý slot chặt chẽ và quy trình đặt chỗ rõ ràng
- **Cải thiện trải nghiệm người dùng** thông qua đặt chỗ nhanh hơn, thông tin minh bạch hơn và thông báo kịp thời hơn
- **Tối ưu vận hành** nhờ giảm đáng kể khối lượng công việc thủ công trong việc điều phối, lập hóa đơn và theo dõi thanh toán
- **Bảo vệ doanh thu** thông qua việc kiểm soát thanh toán, xử lý quá giờ và thực thi chính sách vi phạm
- **Tạo nền tảng mở rộng bền vững** cho mô hình nhiều trạm sạc và lưu lượng người dùng tăng cao
- **Nâng cao khả năng giữ chân khách hàng** nhờ chương trình điểm thưởng và trải nghiệm dịch vụ nhất quán
- **Tăng độ sẵn sàng hạ tầng** nhờ kiến trúc tách biệt rõ frontend, backend và cơ sở dữ liệu

### 3. Kiến trúc giải pháp

Nền tảng được thiết kế theo kiến trúc đơn khối phân lớp, trong đó lớp phân phối frontend, lớp xử lý API backend và lớp cơ sở dữ liệu được tách biệt rõ ràng. Theo sơ đồ triển khai hiện tại, người dùng cuối truy cập giao diện thông qua CloudFront, nơi phân phối nội dung tĩnh được lưu trữ trên Amazon S3. Các yêu cầu API được chuyển đến Nginx và sau đó định tuyến nội bộ đến ứng dụng Spring Boot xử lý nghiệp vụ. Dữ liệu giao dịch được lưu trữ an toàn trong Amazon RDS for SQL Server ở private subnet, trong khi AWS Map hỗ trợ các tính năng hiển thị và tích hợp bản đồ trên frontend.

![Kiến trúc hệ thống trạm sạc EV](/images/2-Proposal/aws-ev-architecture.png)

| Thành phần | Dịch vụ / Công nghệ |
| --------------------- | -------------------------- |
| Lưu trữ frontend | Amazon S3 |
| Phân phối nội dung | Amazon CloudFront |
| Tích hợp bản đồ | AWS Map |
| Bộ định tuyến ngược | Nginx |
| Khung phát triển backend | Spring Boot 3.5.6 |
| Ngôn ngữ lập trình | Java 17 |
| Lớp giao tiếp API | REST Controllers |
| Lớp nghiệp vụ | Service Interfaces và Implementations |
| Lớp truy cập dữ liệu | Spring Data JPA + Hibernate |
| Cơ sở dữ liệu | Amazon RDS for SQL Server |
| Xác thực và phân quyền | Spring Security + JWT + OAuth2 |
| Cổng thanh toán | VNPay |
| Thông báo email | SMTP + Thymeleaf |
| Tài liệu API | Swagger / OpenAPI |
| Xử lý nền | Spring Scheduling + Async Executors |

#### Các dịch vụ AWS được sử dụng

- **Amazon S3**: Lưu trữ tài nguyên tĩnh của frontend và các tệp tải lên khi cần
- **Amazon CloudFront**: Phân phối nội dung frontend và tệp tĩnh với độ trễ thấp
- **Amazon RDS for SQL Server**: Cung cấp hạ tầng cơ sở dữ liệu quan hệ được quản lý trong lớp mạng riêng an toàn hơn
- **AWS Map**: Hỗ trợ hiển thị vị trí và các tính năng bản đồ trên giao diện người dùng
- **Amazon EC2 hoặc môi trường tính toán tương đương**: Lưu trữ Nginx và ứng dụng Spring Boot trong lớp backend
- **Hạ tầng email SMTP**: Phục vụ gửi OTP, xác nhận booking, cập nhật thanh toán và các thông báo vận hành khác

#### Thiết kế các thành phần

Giải pháp được chia thành các lớp và phân hệ chính như sau:

- **Lớp frontend và CDN**: Người dùng cuối truy cập ứng dụng web thông qua CloudFront, nơi phân phối các tài nguyên frontend được lưu trên Amazon S3
- **Lớp tích hợp bản đồ**: Frontend kết nối với AWS Map để lấy và hiển thị dữ liệu bản đồ
- **Lớp tiếp nhận API qua Nginx**: Nginx nhận các yêu cầu HTTPS và chuyển tiếp an toàn đến ứng dụng backend
- **Lớp ứng dụng backend**: Spring Boot xử lý toàn bộ nghiệp vụ như xác thực, booking, phiên sạc, hóa đơn và thanh toán
- **Lớp cơ sở dữ liệu**: Amazon RDS for SQL Server lưu trữ dữ liệu giao dịch như người dùng, booking, hóa đơn, phiên sạc và bản ghi tuân thủ
- **Phân hệ xác thực**: Xử lý đăng ký, đăng nhập, phát hành JWT, vô hiệu hóa token khi đăng xuất, đăng nhập OAuth2 và đặt lại mật khẩu bằng OTP
- **Phân hệ đặt chỗ**: Quản lý giữ slot, tạo booking, xác nhận booking, hủy booking và giải phóng các booking quá hạn
- **Phân hệ phiên sạc**: Điều phối thời điểm bắt đầu và kết thúc phiên sạc, theo dõi điện năng tiêu thụ và trạng thái phiên
- **Phân hệ hóa đơn và tính phí**: Tự động sinh hóa đơn từ các phiên sạc hoàn tất và áp dụng giảm giá khi phù hợp
- **Phân hệ thanh toán**: Tạo đường dẫn thanh toán VNPay, xác thực callback và cập nhật trạng thái giao dịch, hóa đơn
- **Phân hệ trạm sạc và slot**: Quản lý trạm, cổng sạc, biểu phí, lịch vận hành và tình trạng slot theo ngày
- **Phân hệ tuân thủ**: Theo dõi vi phạm của tài xế, áp dụng quy tắc xử lý và hỗ trợ logic cấm tạm thời khi vi phạm lặp lại
- **Phân hệ điểm thưởng**: Quản lý số dư điểm thưởng và hỗ trợ quy đổi điểm khi thanh toán
- **Phân hệ thông báo**: Gửi email bất đồng bộ và hỗ trợ giao tiếp nội bộ theo cơ chế sự kiện

### 4. Triển khai kỹ thuật

#### Cách tiếp cận xử lý nghiệp vụ

Mặc dù hệ thống không sử dụng bộ máy gợi ý dựa trên trí tuệ nhân tạo, nền tảng này áp dụng một cơ chế xử lý nghiệp vụ theo luật, tập trung vào phân bổ slot, tính biểu phí, xác thực thanh toán và thực thi chính sách quản lý.

Các luồng kỹ thuật và nghiệp vụ chính bao gồm:

- Chỉ cho phép tạo booking đối với những slot đang ở trạng thái khả dụng
- Khi booking được xác nhận, các slot liên quan sẽ được khóa cho mục đích sử dụng thực tế
- Khi phiên sạc kết thúc, hệ thống tự động tạo hóa đơn dựa trên dữ liệu sử dụng
- Điểm thưởng có thể được áp dụng trong bước thanh toán và được chốt sau khi giao dịch thành công
- Cơ chế nhóm vi phạm hỗ trợ nâng mức xử lý và ban tạm thời tài xế khi cần
- Các tác vụ chạy nền theo lịch sẽ tự động sinh slot mới và giải phóng booking quá hạn
- Nginx chuyển tiếp lưu lượng HTTPS an toàn đến backend để xử lý nội bộ
- Backend kết nối với Amazon RDS for SQL Server để thực hiện các truy vấn dữ liệu trong hạ tầng được kiểm soát

Cách tiếp cận này giúp hệ thống đảm bảo tính nhất quán, khả năng kiểm soát và độ tin cậy cao trong toàn bộ quy trình vận hành.

#### Yêu cầu kỹ thuật

Giải pháp backend được xây dựng trên tập hợp công nghệ và yêu cầu kỹ thuật như sau:

- **Phân phối frontend**: Tài nguyên tĩnh của frontend được lưu trên Amazon S3 và phân phối qua Amazon CloudFront
- **Tích hợp bản đồ**: AWS Map phục vụ hiển thị bản đồ và các chức năng liên quan đến vị trí
- **Định tuyến API**: Nginx dùng để tiếp nhận và chuyển tiếp lưu lượng API
- **Khung phát triển backend**: Spring Boot với Java 17 và Maven
- **Bảo mật**: Spring Security, JWT, OAuth2 và BCrypt để mã hóa mật khẩu
- **Cơ sở dữ liệu**: Amazon RDS for SQL Server kết hợp Hibernate và Spring Data JPA
- **Thanh toán**: Tích hợp VNPay với cơ chế xác thực chữ ký HMAC-SHA512
- **Thông báo**: Dịch vụ email SMTP với template Thymeleaf
- **Xử lý bất đồng bộ**: Thread pool riêng kết hợp `@Async` để tránh chặn luồng xử lý chính
- **Tác vụ theo lịch**: Các job theo cron hoặc fixed delay để quản lý slot và booking
- **Tài liệu kỹ thuật**: Swagger / OpenAPI phục vụ việc kiểm thử và tích hợp API

### 5. Kế hoạch triển khai và các cột mốc

Dự án có thể được triển khai theo từng giai đoạn rõ ràng để giảm rủi ro và đảm bảo kiểm soát chất lượng:

- **Tuần 7-8**: Hoàn thiện mô hình dữ liệu, cấu hình bảo mật, cấu trúc cơ sở dữ liệu, bố cục hạ tầng frontend-backend và cấu hình cloud cốt lõi
- **Tuần 8**: Xây dựng các luồng booking, vòng đời phiên sạc và quản lý trạm-slot
- **Tuần 9**: Tích hợp sinh hóa đơn, cổng thanh toán VNPay, dịch vụ thông báo email và phần tích hợp bản đồ
- **Tuần 10**: Hoàn thiện tính năng điểm thưởng, quản lý vi phạm và các chức năng theo vai trò
- **Tuần 11**: Thực hiện kiểm thử tích hợp, tối ưu hiệu năng, xác thực hạ tầng và hardening cho môi trường production
- **Tuần 12**: Triển khai hệ thống, thiết lập giám sát và chuẩn bị đưa vào vận hành chính thức

Cách triển khai theo từng mốc như trên giúp mỗi phân hệ được hoàn thiện và kiểm thử trước khi chuyển sang bước tiếp theo.

### 6. Dự toán chi phí

| Thành phần | Dịch vụ | Chi phí ước tính |
| -------------------- | -------------------------- | -------------- |
| Lưu trữ frontend | Amazon S3 | Thấp |
| Phân phối nội dung | CloudFront | Thấp |
| Hạ tầng backend | EC2 hoặc môi trường chạy tương đương | Trung bình |
| Cơ sở dữ liệu | Amazon RDS for SQL Server | Trung bình đến cao |
| Dịch vụ bản đồ | AWS Map | Phụ thuộc mức sử dụng |
| Dịch vụ email | SMTP / nhà cung cấp email | Thấp |
| Cổng thanh toán | Phí giao dịch VNPay | Thay đổi theo sản lượng giao dịch |
| Giám sát và nhật ký hệ thống | Hệ thống theo dõi cloud tùy chọn | Thấp đến trung bình |
| **Tổng cộng** |  | **Phụ thuộc mức sử dụng thực tế** |

**Lưu ý**: Chi phí thực tế sẽ phụ thuộc vào mô hình triển khai, lưu lượng truy cập, số lượng trạm sạc, tần suất sử dụng API, tần suất thanh toán, số lượng yêu cầu bản đồ và dung lượng lưu trữ phát sinh. Với kiến trúc hiện tại, hệ thống phù hợp để triển khai theo từng giai đoạn với chi phí ban đầu hợp lý, sau đó mở rộng theo nhu cầu thực tế.

### 7. Đánh giá rủi ro

#### Ma trận rủi ro

Một số rủi ro quan trọng đã được xác định trong thiết kế và triển khai hiện tại gồm:

- **Xung đột khi nhiều người cùng đặt một slot**: Tác động cao, xác suất trung bình
- **Lỗi callback hoặc đối soát thanh toán**: Tác động cao, xác suất trung bình
- **Thông tin bí mật lưu trong tệp cấu hình**: Tác động cao, xác suất cao
- **Thiếu logging và khả năng quan sát hệ thống**: Tác động trung bình, xác suất trung bình
- **Độ phủ kiểm thử tự động còn hạn chế**: Tác động trung bình, xác suất trung bình
- **Nguy cơ nghẽn hiệu năng cơ sở dữ liệu khi mở rộng**: Tác động trung bình, xác suất trung bình
- **Nguy cơ cấu hình sai giữa các lớp frontend, backend và cơ sở dữ liệu**: Tác động trung bình, xác suất trung bình

#### Chiến lược giảm thiểu

Để giảm thiểu các rủi ro trên, cần triển khai các biện pháp sau:

- Tăng cường ràng buộc giao dịch và constraint ở cơ sở dữ liệu cho luồng giữ slot
- Chuyển secrets sang cơ chế quản lý tập trung trong môi trường production
- Cải thiện xử lý ngoại lệ và xây dựng chiến lược phản hồi lỗi thống nhất cho API
- Mở rộng phạm vi kiểm thử unit, integration và end-to-end
- Thiết lập logging có cấu trúc, tracing theo request và dashboard giám sát
- Tối ưu truy vấn dữ liệu thông qua index, phân trang và tuning query
- Bổ sung rate limiting và cơ chế chống lạm dụng đối với các endpoint xác thực và thanh toán
- Kiểm tra kỹ cấu hình mạng, subnet, HTTPS routing và chính sách kết nối từ backend đến database trước khi đưa vào production

#### Phương án dự phòng

Trong trường hợp phát sinh sự cố trong quá trình vận hành, hệ thống cần chuẩn bị các phương án dự phòng như sau:

- **Sự cố thanh toán**: Cho phép xác minh lại giao dịch và hỗ trợ quy trình đối soát thủ công
- **Lỗi gửi email**: Bổ sung cơ chế retry và theo dõi trạng thái thông báo
- **Lỗi scheduler hoặc sinh slot**: Cung cấp job khôi phục thủ công và dashboard hỗ trợ vận hành
- **Vấn đề hiệu năng khi tăng trưởng nhanh**: Áp dụng cache, tối ưu các endpoint đọc nhiều và chuẩn bị cho phương án tách dịch vụ nếu cần
- **Rủi ro lộ thông tin bảo mật**: Xoay vòng thông tin truy cập ngay lập tức và chuyển hoàn toàn sang secret manager
- **Sự cố hạ tầng triển khai**: Chuẩn bị quy trình rollback và checklist xác thực môi trường

### 8. Kết quả kỳ vọng

#### Cải tiến kỹ thuật

Giải pháp backend được kỳ vọng mang lại các cải tiến kỹ thuật rõ rệt như:

- Tập trung toàn bộ điều phối booking, phiên sạc, hóa đơn và thanh toán vào một nền tảng duy nhất
- Tăng độ ổn định nhờ các luật nghiệp vụ rõ ràng, tự động hóa và tác vụ bảo trì theo lịch
- Đảm bảo phân quyền và bảo mật tốt cho tài xế, nhân viên và quản trị viên
- Rút ngắn thời gian xử lý thanh toán số và đối soát hóa đơn
- Nâng cao khả năng phân phối frontend thông qua Amazon S3 và CloudFront
- Tăng độ an toàn cho xử lý backend thông qua lớp Nginx và ứng dụng Spring Boot
- Tăng mức độ tách biệt hạ tầng nhờ cơ sở dữ liệu được đặt trên Amazon RDS for SQL Server ở lớp riêng
- Tăng khả năng bảo trì hệ thống nhờ tách lớp rõ ràng giữa controller, service và repository

#### Giá trị dài hạn

Về dài hạn, nền tảng không chỉ giải quyết bài toán kỹ thuật trước mắt mà còn tạo ra giá trị chiến lược cho đơn vị vận hành:

- **Khả năng mở rộng vận hành**: Hỗ trợ phát triển mô hình nhiều trạm và số lượng booking ngày càng tăng
- **Sẵn sàng thương mại hóa**: Tạo nền tảng số cho việc khai thác doanh thu từ dịch vụ sạc EV
- **Nâng cao khả năng giữ chân khách hàng**: Khuyến khích sử dụng lặp lại thông qua điểm thưởng và trải nghiệm mượt mà hơn
- **Tăng cường kiểm soát chính sách**: Quản lý tốt hơn các trường hợp quá giờ, sử dụng sai quy định và vi phạm lặp lại
- **Dễ mở rộng trong tương lai**: Có thể phát triển sang kiến trúc microservices, phân tích nâng cao, tích hợp di động và dashboard vận hành
- **Tạo nền tảng dữ liệu kinh doanh**: Dữ liệu booking, phiên sạc và thanh toán có thể phục vụ dự báo, phân tích hiệu suất và tối ưu chiến lược giá trong tương lai
- **Sẵn sàng về mặt hạ tầng**: Thiết lập được mô hình cloud gần với production hơn, với ranh giới rõ ràng giữa lớp phân phối, lớp ứng dụng và lớp dữ liệu