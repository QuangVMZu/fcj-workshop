---
title: "Thực thi phiên sạc"
date: 2025-10-13
weight: 4
chapter: false
pre: " <b> 4.4. </b> "
---

## Tổng quan luồng

Sau bước xác nhận booking, hệ thống chuyển sang giai đoạn thực thi phiên sạc. Phần này kết hợp màn hình start session của staff, giao diện charging phía driver, backend scheduling và transaction boundary dùng để hoàn tất session và sinh invoice.

## Các điểm implementation quan trọng

- staff có thể bắt đầu session từ các màn hình quản lý phiên sạc, trong khi driver nhìn thấy charging view ở phía client,
- backend chỉ khởi động session từ booking `CONFIRMED` còn nằm trong khung giờ hợp lệ,
- SOC và tiến độ charging được mô phỏng thay vì lấy từ phần cứng,
- tiến độ session được phản ánh cả ở backend status lẫn temporary state trong trình duyệt,
- stop logic tính chi phí và tạo invoice cho bước payment phía sau.

## Nội dung

1. [Bắt đầu phiên sạc](4.4.1-start-session/)
2. [Giao diện charging và SOC mô phỏng](4.4.2-live-status-and-simulated-soc/)
3. [Luồng dừng sạc, auto-stop và tính chi phí](4.4.3-stop-session-and-costing/)
4. [Xác thực luồng charging và các rủi ro](4.4.4-charging-validation-and-risks/)
