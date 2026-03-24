---
title: "Đăng ký, đăng nhập và trạng thái xác thực phía client"
date: 2025-10-13
weight: 1
chapter: false
pre: " <b> 4.3.1. </b> "
---

## Điểm vào xác thực ở frontend

Frontend bắt đầu bằng:

- `Register.jsx` gọi `registerApi()`,
- `Verify.jsx` gọi `verifyOtp()`,
- `Login.jsx` gọi `useLogin()` rồi `loginApi()` đến `POST /api/users/login`.

Khi thành công, client lưu `accessToken`, `role` và `userDetails` vào `localStorage` và Redux. Nếu role đăng nhập là `STAFF`, frontend còn gọi thêm API để lấy station assignment trước khi điều hướng sang các màn hình của staff.

## Auth path ở backend

Luồng phía backend là:

`UsersController.login()` -> `UserServiceImpl.authenticate()` -> `TokenService.generateToken()`

Sau đó `JwtAuthFilter` sẽ bảo vệ các request tiếp theo.

## Ý nghĩa của client state

Luồng này rất quan trọng vì các màn hình booking, charging, payment và notification về sau đều phụ thuộc vào client state đã lưu trước đó. Implementation hiện tại cũng phản ánh mức độ tin cậy khá cao vào auth state nằm ở phía client, điều này sẽ xuất hiện lại trong phần limitations.
