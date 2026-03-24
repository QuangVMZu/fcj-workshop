---
title: "Các quan sát implementation và rủi ro đã biết"
date: 2025-10-13
weight: 2
chapter: false
pre: " <b> 4.6.2. </b> "
---

## Quan sát ở phía backend

- secrets đang bị commit trong `application.properties`,
- hành vi authorization chưa thật sự nhất quán,
- việc giữ booking vẫn có rủi ro race condition,
- booking `PENDING` đã chặn công suất,
- việc hủy booking quá hạn có thể không giải phóng hết slot đã giữ,
- charging telemetry là mô phỏng,
- trạng thái invoice và VNPay có thể lệch nhau,
- violation flow chưa khép kín hoàn toàn,
- phần nối OAuth2 còn thiếu đồng đều,
- độ phủ test hiện rất thấp.

## Quan sát ở phía frontend

- UI authorization tin quá nhiều vào client state và hành vi `?token=`,
- luồng stop-session phụ thuộc vào `sessionStorage` dùng chung giữa các thiết bị,
- OTP resend chỉ mới được mô phỏng ở UI,
- payment retry navigation đang bị lỗi,
- logic OAuth callback bị lặp,
- ý nghĩa trạng thái của charging point còn thiếu nhất quán,
- một số đoạn UI code còn mong manh, bao gồm lỗi import và xử lý response shape không ổn định.

## Cách diễn giải

Các vấn đề này không làm Workshop mất giá trị. Chúng cho thấy mức độ trưởng thành thực tế của implementation hiện tại và cần được ghi nhận thẳng thắn thay vì che đi.
