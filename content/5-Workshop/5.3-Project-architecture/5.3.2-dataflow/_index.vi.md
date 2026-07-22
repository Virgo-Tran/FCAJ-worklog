---
title : "Luồng dữ liệu chi tiết"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

Luồng xử lý chính của hệ thống gồm các bước sau:

**Bước 1 — Phân giải DNS.** Route 53 phân giải `airport.example.com` và chuyển hướng người dùng đến CloudFront.

**Bước 2 — Phục vụ giao diện.** CloudFront phục vụ giao diện web quản lý sân bay (Airport Web UI) từ S3 Static Frontend.

**Bước 3 — Xác thực.** Người dùng (hành khách/nhân viên) xác thực qua Cognito và nhận JWT.

**Bước 4 — Định tuyến API.** CloudFront/ALB định tuyến các request API (quản lý chuyến bay, hành khách...) đến ECS Cluster.

**Bước 5 — Phân quyền.** ECS Cluster xác thực JWT trực tiếp với Cognito, đồng thời gọi **Lambda Get OwnerID Service** (Lambda này tra cứu Cognito Admin API) để lấy Owner ID gắn với request.

**Bước 6 — Dữ liệu nghiệp vụ.** ECS Cluster đọc/ghi dữ liệu nghiệp vụ (chuyến bay, hành khách) vào RDS PostgreSQL.

**Bước 7 — Tải tài liệu lên.** ECS Cluster sinh Presigned URL (qua S3 Gateway Endpoint); nhân viên/hành khách tải boarding pass hoặc thẻ hành lý trực tiếp lên S3 Airport Documents.

**Bước 8 — Kích hoạt OCR.** S3 phát sự kiện `Object Created` → **SQS Document Processing** → **Lambda Submit Job** → gọi Amazon Textract để OCR (trích xuất tên, số hiệu chuyến bay, số ghế, cổng, hộ chiếu…).

**Bước 9 — Xử lý kết quả OCR.** Textract hoàn tất, đẩy kết quả vào **SQS OCR Results**; **Lambda Process OCR Result** đọc hàng đợi, ghi dữ liệu trích xuất vào RDS, rồi gửi thông báo qua SES (qua VPC Endpoint).

**Bước 10 — Thông báo định kỳ.** EventBridge kích hoạt định kỳ hàng tháng → **Lambda Process Scheduled Tasks** truy vấn RDS lấy dữ liệu chuyến bay → gửi thông báo hàng loạt qua SES.

**Bước 11 — Xử lý lỗi.** Message xử lý lỗi (quá số lần thử) rơi vào **Dead-Letter Queue** → CloudWatch cảnh báo để đội vận hành kiểm tra và xử lý thủ công.
