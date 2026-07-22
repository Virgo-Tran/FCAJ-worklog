---
title : "Danh sách dịch vụ AWS sử dụng"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

Dự án sử dụng **18 dịch vụ AWS**, áp dụng đúng công nghệ nền tảng của kiến trúc mẫu V2, tùy biến vai trò cho nghiệp vụ quản lý sân bay:

| STT | Dịch vụ AWS | Nhóm dịch vụ | Vai trò trong kiến trúc |
|---|---|---|---|
| 1 | **Route 53** | Networking & Content Delivery | Phân giải tên miền `airport.example.com`, đưa người dùng (hành khách/nhân viên) đến CloudFront gần nhất. |
| 2 | **CloudFront** | Networking & Content Delivery | Phân phối giao diện web quản lý sân bay (Airport Web UI) và định tuyến các request API động xuống ALB. |
| 3 | **WAF** | Security, Identity & Compliance | Bảo vệ hệ thống trước các cuộc tấn công web khi tiếp nhận lượng lớn truy cập từ hành khách. |
| 4 | **Amazon S3** | Storage | Lưu giao diện web tĩnh (S3 Static Frontend) và lưu tài liệu do hành khách/nhân viên tải lên: thẻ lên máy bay (boarding pass), thẻ hành lý (luggage tag). |
| 5 | **Amazon Cognito** | Security, Identity & Compliance | Xác thực nhân viên/hành khách sân bay, phát hành JWT; đồng thời cung cấp Owner ID gắn với từng tài liệu tải lên. |
| 6 | **AWS Lambda** | Compute (Serverless) | Ba hàm phục vụ ba nghiệp vụ riêng biệt: **Get OwnerID Service** (tra cứu chủ sở hữu), **Lambda Submit Job** (kích hoạt OCR), **Lambda Process OCR Result & Lambda Process Scheduled Tasks** (ghi kết quả OCR, gửi thông báo định kỳ). |
| 7 | **Amazon VPC, IGW & VPC Endpoint** | Networking & Content Delivery | Cô lập mạng cho toàn bộ dịch vụ nghiệp vụ sân bay; VPC Endpoint cho phép ECS/Lambda truy cập S3 và Textract/SES an toàn mà không cần ra Internet công cộng. |
| 8 | **Application Load Balancer** | Networking & Content Delivery | Phân phối các request API quản lý chuyến bay/hành khách đến các Fargate Task trên 2 Availability Zone. |
| 9 | **Amazon ECS trên AWS Fargate** | Compute (Container) | Chạy các API nghiệp vụ sân bay (quản lý chuyến bay, hành khách, boarding pass) dạng container, tự động mở rộng theo lưu lượng giờ cao điểm. |
| 10 | **Amazon RDS (PostgreSQL)** | Database | Lưu dữ liệu quan hệ: sân bay, chuyến bay, hành khách, boarding pass, tài liệu, thông báo — với Standby để dự phòng. |
| 11 | **Amazon SQS** | Application Integration | **SQS Document Processing** (hàng đợi xử lý tài liệu upload), **Dead-Letter Queue** (message lỗi), **SQS OCR Results** (kết quả OCR chờ xử lý). |
| 12 | **Amazon Textract** | Machine Learning | Trích xuất tự động: tên hành khách, số hiệu chuyến bay, ngày giờ khởi hành/đến, số ghế, cổng, số hộ chiếu... từ ảnh boarding pass. |
| 13 | **Amazon EventBridge** | Application Integration | Kích hoạt định kỳ hàng tháng để hệ thống tổng hợp và gửi thông báo chuyến bay cho hành khách. |
| 14 | **Amazon SES** | Business Applications | Gửi email thông báo chuyến bay: trễ giờ, đổi cổng, nhắc lên máy bay, huỷ chuyến, thông báo hàng tháng. |
| 15 | **Amazon CloudWatch (+ Logs)** | Management & Governance | Giám sát hoạt động ECS, log tập trung, cảnh báo khi tài liệu xử lý OCR thất bại (rơi vào DLQ). |
| 16 | **AWS CodeBuild** | Developer Tools | Build Docker image cho các service quản lý sân bay từ mã nguồn trên GitHub. |
| 17 | **Amazon ECR** | Developer Tools | Lưu trữ image container của các service sân bay theo từng phiên bản. |
| 18 | **AWS CodeDeploy** | Developer Tools | Triển khai tự động phiên bản mới của service lên ECS Cluster mỗi khi có thay đổi được duyệt. |

{{% notice note %}}
Kiến trúc tham khảo liệt kê 19 dịch vụ trong khi dự án dùng 18 — dự án gộp các mục liên quan đến Lambda thành một dòng duy nhất bao gồm cả ba hàm.
{{% /notice %}}
