---
title : "Luồng dữ liệu chi tiết"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2.2 </b> "
---

Luồng xử lý chính của hệ thống được chia thành các bước sau:

**Bước 1** — Người dùng truy cập tên miền, Route 53 phân giải DNS và chuyển hướng đến CloudFront.

**Bước 2** — CloudFront phục vụ giao diện web tĩnh từ S3 (Static Frontend); WAF kiểm tra và lọc traffic độc hại trước khi đến CloudFront.

**Bước 3** — Người dùng xác thực qua Cognito (nhận JWT); Cognito liên kết với **Lambda Get OwnerID Service** để tra cứu Owner ID.

**Bước 4** — CloudFront chuyển các request API động xuống Application Load Balancer (ALB) trong subnet công khai.

**Bước 5** — ALB phân phối request đến các container ECS chạy trên AWS Fargate, đặt trong private subnet và trải trên 2 AZ.

**Bước 6** — ECS/Fargate đọc/ghi dữ liệu nghiệp vụ vào RDS PostgreSQL (Primary), dữ liệu được đồng bộ sang Standby theo thời gian thực.

**Bước 7** — Người dùng tải tệp lên S3 (Image S3) thông qua Presigned URL do backend cấp; sự kiện `Object Created` được đẩy vào SQS.

**Bước 8** — **Lambda Submit Job** nhận message từ SQS, gọi Amazon Textract (qua VPC Endpoint) để thực hiện OCR trên tài liệu.

**Bước 9** — Textract xử lý xong, đẩy kết quả vào **SQS OCR Results**; **Lambda Process** đọc hàng đợi này và ghi kết quả vào RDS.

**Bước 10** — EventBridge kích hoạt Lambda Process theo lịch định kỳ để gửi thông báo qua SES (qua VPC Endpoint).

**Bước 11** — Toàn bộ hoạt động CI/CD (GitHub → CodeBuild → ECR → CodeDeploy) tự động build, đóng gói và triển khai phiên bản mới lên ECS Cluster; CloudWatch giám sát toàn hệ thống và ghi log tập trung.

{{% notice tip %}}
Toàn bộ luồng dữ liệu được thiết kế khép kín theo đúng nguyên tắc gọi hàm (caller gọi đến callee), và các tài nguyên trong private subnet được kết nối tới S3, Textract, SES thông qua VPC Endpoint — hạn chế tối đa việc đi qua Internet công cộng.
{{% /notice %}}
