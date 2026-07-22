---
title : "Kết luận"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

#### Tổng kết

Kiến trúc của Airport Management System được xây dựng trên nền tảng AWS hiện đại, bám sát mô hình tham khảo trình bày ở [phần kiến trúc tham khảo](../5.2-Reference-architecture/) về cả công nghệ và cách triển khai:

+ **CloudFront kết hợp S3** cho tầng giao diện.
+ **Cognito** cho xác thực người dùng.
+ **ECS/Fargate đứng sau Application Load Balancer** cho tầng API nghiệp vụ.
+ **RDS PostgreSQL triển khai Multi-AZ** cho dữ liệu.
+ **SQS kết hợp Textract và Lambda** cho pipeline xử lý OCR.
+ **EventBridge và SES** cho hệ thống thông báo.
+ **CodeBuild, ECR và CodeDeploy** cho pipeline CI/CD tự động.

Toàn bộ luồng dữ liệu được thiết kế khép kín theo đúng nguyên tắc gọi hàm (caller gọi đến callee), và các tài nguyên trong private subnet được kết nối tới S3, Textract, SES thông qua **VPC Endpoint** theo đúng khuyến nghị bảo mật mạng của AWS — hạn chế tối đa việc đi qua Internet công cộng.

#### Đề xuất mở rộng

{{% notice tip %}}
Các hướng cải tiến trong tương lai:

+ Bổ sung cơ chế giám sát chi phí với **AWS Budgets / Cost Explorer**.
+ Cân nhắc thêm **AWS Backup** cho RDS.
+ Đánh giá thêm nhu cầu **NAT Gateway** cho các tác vụ ngoài phạm vi VPC Endpoint — ví dụ gọi API bên thứ ba nằm ngoài AWS.
{{% /notice %}}
