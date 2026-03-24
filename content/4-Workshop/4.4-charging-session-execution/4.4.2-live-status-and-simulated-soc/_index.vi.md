---
title: "Giao diện charging và SOC mô phỏng"
date: 2025-10-13
weight: 2
chapter: false
pre: " <b> 4.4.2. </b> "
---

## Giao diện live ở frontend

`ChargingSession.jsx` polling backend để lấy session state, sau đó mô phỏng live SOC và energy ngay trong trình duyệt. Trang này lưu `session_{sessionId}_live_soc` vào `sessionStorage` để giao diện charging vẫn phản hồi tốt giữa các lần refresh.

## Hành vi trạng thái ở backend

Backend có trả trạng thái charging, nhưng `getChargingStatus()` hiện vẫn là mô phỏng chứ chưa gắn với telemetry thật từ charger. Đây là giới hạn có chủ đích của implementation, không phải do thiếu tài liệu.

## Hệ quả quan trọng

Vì SOC được duy trì một phần trong browser storage, trải nghiệm charging có thể khác nhau giữa các thiết bị hoặc các user context khác nhau. Hệ thống hiện vận hành như một full-stack demo flow với việc phối hợp state ở mức ứng dụng, chứ chưa phải một nền tảng charging lấy thiết bị làm nguồn sự thật tuyệt đối.
