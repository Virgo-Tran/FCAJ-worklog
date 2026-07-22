---
title : "Kiến trúc tham khảo (AWS V2)"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### AWS V2 là gì

**AWS V2** là bản kiến trúc tham khảo được dùng làm nền tảng thiết kế cho Hệ thống Quản lý Thông tin Sân bay. Đây là một kiến trúc web ba lớp (three-tier) hiện đại trên AWS, kết hợp mô hình **serverless** (Lambda, S3, CloudFront) cho các tác vụ sự kiện và mô hình **container hoá** (ECS/Fargate) cho tầng API nghiệp vụ chính, cùng với một pipeline xử lý tài liệu bằng OCR (Amazon Textract) và một pipeline CI/CD tự động hoá triển khai.

Kiến trúc được tổ chức theo mô hình chuẩn của **AWS Well-Architected Framework**: tách biệt public subnet (chứa ALB, VPC Endpoint) và private subnet (chứa container ứng dụng và cơ sở dữ liệu), triển khai trên tối thiểu 2 Availability Zone (AZ-1, AZ-2) để đảm bảo tính sẵn sàng cao (High Availability).

#### Sơ đồ kiến trúc tổng quan

![Kiến trúc tham khảo AWS V2](/images/5-Workshop/aws-v2-reference-architecture.png)

*Hình 1.1 — Sơ đồ kiến trúc tham khảo AWS V2, bao gồm Route 53 và pipeline DEV/CI-CD.*

#### Nội dung

- [Danh sách dịch vụ AWS sử dụng](5.2.1-services/)
- [Luồng dữ liệu chi tiết](5.2.2-dataflow/)
- [Pipeline CI/CD](5.2.3-cicd/)
