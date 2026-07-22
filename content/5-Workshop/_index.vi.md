---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Hệ thống Quản lý Thông tin Sân bay trên AWS

#### Tổng quan

Workshop này trình bày kiến trúc của **Hệ thống Quản lý Thông tin Sân bay** — hệ thống phục vụ hai nhóm người dùng chính là hành khách và nhân viên sân bay, cho phép xác thực người dùng, quản lý thông tin chuyến bay và hành khách, tự động trích xuất dữ liệu từ thẻ lên máy bay và thẻ hành lý bằng công nghệ OCR, đồng thời gửi thông báo tự động về tình trạng chuyến bay.

Hệ thống được xây dựng theo mô hình **kiến trúc web ba lớp (three-tier)** hiện đại trên AWS, kết hợp hai mô hình triển khai:

+ **Serverless** (Lambda, S3, CloudFront) cho các tác vụ xử lý theo sự kiện.
+ **Container** (ECS trên AWS Fargate) cho tầng API nghiệp vụ chính.

Bên trên hai mô hình này là một **pipeline xử lý tài liệu OCR** dựa trên Amazon Textract và một **pipeline CI/CD** tự động hoá việc triển khai.

Kiến trúc được tổ chức theo chuẩn **AWS Well-Architected Framework**: tách biệt public subnet (chứa ALB, VPC Endpoint) và private subnet (chứa container ứng dụng và cơ sở dữ liệu), triển khai trên tối thiểu **2 Availability Zone** (AZ-1, AZ-2) để đảm bảo tính sẵn sàng cao.

#### Nội dung

1. [Giới thiệu](5.1-Introduction/)
2. [Kiến trúc tham khảo (AWS V2)](5.2-Reference-architecture/)
3. [Kiến trúc hệ thống sân bay](5.3-Project-architecture/)
4. [Pipeline xử lý tài liệu OCR](5.4-OCR-pipeline/)
5. [Thiết kế cơ sở dữ liệu](5.5-Database-design/)
6. [Chạy thử prototype](5.6-Prototype/)
7. [Kết luận](5.7-Conclusion/)
