---
title : "Danh sách dịch vụ AWS sử dụng"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.2.1 </b> "
---

Kiến trúc mẫu sử dụng tổng cộng **19 dịch vụ AWS** thuộc nhiều nhóm khác nhau: Networking & Content Delivery, Security, Compute, Database, Application Integration, Machine Learning, Management & Governance, và Developer Tools (CI/CD).

| STT | Dịch vụ AWS | Nhóm dịch vụ | Vai trò trong kiến trúc |
|---|---|---|---|
| 1 | **Route 53** | Networking & Content Delivery | Dịch vụ DNS, phân giải tên miền (vd. `airport.example.com`) và định tuyến người dùng đến CloudFront ở tầng biên. |
| 2 | **CloudFront** | Networking & Content Delivery | CDN toàn cầu, là điểm vào (entry point) duy nhất cho mọi truy cập; cache và phân phối giao diện web tĩnh, đồng thời chuyển tiếp các request API động xuống ALB. |
| 3 | **WAF** | Security, Identity & Compliance | Gắn vào CloudFront để lọc và chặn các cuộc tấn công web phổ biến (SQL Injection, XSS, bot rà quét) trước khi request đến được backend. |
| 4 | **Amazon S3** | Storage | Lưu trữ mã nguồn tĩnh của giao diện web (Static Frontend) và lưu file người dùng tải lên (Image S3) thông qua Presigned URL. |
| 5 | **Amazon Cognito** | Security, Identity & Compliance | Quản lý danh tính người dùng, xác thực và phát hành JSON Web Token (JWT) dùng cho các lời gọi API tiếp theo. |
| 6 | **AWS Lambda** | Compute (Serverless) | Chạy các hàm xử lý sự kiện không cần quản lý máy chủ: tra cứu Owner ID, xử lý job OCR (Submit Job), và các tác vụ định kỳ (Process). |
| 7 | **Amazon VPC & Internet Gateway** | Networking & Content Delivery | Mạng riêng ảo cô lập toàn bộ tài nguyên backend; Internet Gateway cho phép các tài nguyên ở subnet công khai (ALB) giao tiếp với Internet. |
| 8 | **VPC Endpoint** (Gateway & Interface) | Networking & Content Delivery | Cho phép tài nguyên nằm trong private subnet (ECS, Lambda) gọi S3, Textract, SES trực tiếp qua mạng nội bộ AWS, không cần NAT Gateway hay đi qua Internet. |
| 9 | **Elastic Load Balancing (ALB)** | Networking & Content Delivery | Application Load Balancer phân phối các request API đến các container đang chạy trên nhiều Availability Zone, đảm bảo tính sẵn sàng cao. |
| 10 | **Amazon ECS trên AWS Fargate** | Compute (Container) | Chạy service backend dạng container theo mô hình serverless container (không quản lý EC2), tự động mở rộng theo tải, đặt trong private subnet. |
| 11 | **Amazon RDS (PostgreSQL)** | Database | Cơ sở dữ liệu quan hệ chính, triển khai theo mô hình Primary/Standby (Multi-AZ) để đảm bảo tính sẵn sàng cao và tự động chuyển đổi khi có sự cố. |
| 12 | **Amazon SQS** | Application Integration | Hàng đợi thông điệp trung gian: tách rời (decouple) việc tải tài liệu lên khỏi quá trình xử lý OCR, có Dead-Letter Queue riêng cho message lỗi và một hàng đợi riêng cho kết quả OCR. |
| 13 | **Amazon Textract** | Machine Learning | Dịch vụ AI trích xuất văn bản và dữ liệu có cấu trúc (OCR) từ ảnh tài liệu được tải lên. |
| 14 | **Amazon EventBridge** | Application Integration | Lập lịch (giống cron) để tự động kích hoạt các tác vụ định kỳ, ví dụ gửi thông báo hàng tháng. |
| 15 | **Amazon SES** | Business Applications | Gửi email thông báo tới người dùng cuối (kết quả xử lý, cảnh báo, thông báo định kỳ). |
| 16 | **Amazon CloudWatch (+ Logs)** | Management & Governance | Thu thập log tập trung, theo dõi chỉ số (metrics) và cảnh báo khi có message rơi vào Dead-Letter Queue hoặc lỗi hệ thống. |
| 17 | **AWS CodeBuild** | Developer Tools | Biên dịch mã nguồn và đóng gói (build) Docker image cho service backend. |
| 18 | **Amazon ECR** | Developer Tools | Lưu trữ và quản lý phiên bản Docker image đã build, làm nguồn image để CodeDeploy triển khai. |
| 19 | **AWS CodeDeploy** | Developer Tools | Tự động triển khai phiên bản container mới nhất từ ECR lên ECS Cluster, hỗ trợ rolling deployment. |
