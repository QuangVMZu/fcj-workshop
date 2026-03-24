---
title: "Luồng dừng sạc, auto-stop và tính chi phí"
date: 2025-10-13
weight: 3
chapter: false
pre: " <b> 4.4.3. </b> "
---

## Các điểm vào của luồng dừng

Luồng stop có thể được kích hoạt bởi:

- endpoint dừng sạc của driver,
- endpoint dừng sạc của staff,
- auto-stop được lên lịch tại thời điểm kết thúc booking.

Các charging view ở frontend cũng cố gắng đọc lại SOC đã lưu trong browser khi thao tác dừng sạc diễn ra.

## Transaction boundary

Tất cả các đường dừng sạc cuối cùng đều đi qua `ChargingSessionTxHandler.stopSessionInternalTx()`.

Handler mang tính transaction này sẽ tính:

- thời lượng,
- charging point và connector context,
- estimated final SOC,
- energy,
- tariff và cost,
- overstay tính theo số phút vượt quá thời điểm ước lượng để đạt 100 phần trăm SOC.

Khi thành công, session được hoàn tất, booking được hoàn tất, notification được tạo, và một `Invoice(UNPAID)` được sinh ra cho bước payment phía sau.
