---
title: "Điểm vào của invoice và payment"
date: 2025-10-13
weight: 1
chapter: false
pre: " <b> 4.5.1. </b> "
---

## Điểm vào từ invoice

Luồng payment bắt đầu từ invoice được tạo bởi `ChargingSessionTxHandler.stopSessionInternalTx()`. Invoice này là nguồn sự thật thương mại cho bước xử lý kế tiếp.

## Màn hình payment ở frontend

`Payment.jsx` tải:

- chi tiết invoice,
- payment methods,
- discount information,
- loyalty context.

Frontend cũng ẩn các lựa chọn `VNPAY` và `EWALLET` khi số tiền phải trả thấp hơn `10000`, nên UI đã định hình đường đi của payment trước cả khi backend xử lý nó.

## Điểm vào payment ở backend

Ở phía server, payment logic rẽ ra từ `VnPayController.create()` và các service thanh toán liên quan.
