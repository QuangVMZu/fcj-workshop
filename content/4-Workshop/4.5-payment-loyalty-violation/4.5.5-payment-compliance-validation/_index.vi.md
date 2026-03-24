---
title: "Xác thực payment và compliance"
date: 2025-10-13
weight: 5
chapter: false
pre: " <b> 4.5.5. </b> "
---

## Danh sách xác thực

Cần xác nhận rằng:

- trạng thái invoice, transaction và trang payment result ở frontend vẫn nhất quán với nhau,
- direct payment và VNPay đều dẫn tới cùng một kết quả thương mại,
- discount được áp dụng trước khi settlement hoàn tất,
- loyalty thay đổi đúng theo quy tắc dùng điểm hay cộng điểm,
- booking quá hạn có thể tạo violation và triplet,
- vi phạm lặp lại có thể leo thang thành driver ban.

## Các điểm ma sát đã biết

- trạng thái invoice và VNPay có thể lệch nhau,
- payment retry navigation ở frontend còn mong manh,
- việc xử lý response shape ở nhiều client API flow hiện chưa ổn định.
