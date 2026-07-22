---
title : "Kiến trúc hệ thống sân bay"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Áp dụng kiến trúc tham khảo

Hệ thống Quản lý Thông tin Sân bay được xây dựng dựa trên kiến trúc mẫu AWS V2 ở [phần trước](../5.2-Reference-architecture/), áp dụng cùng công nghệ nền tảng — ECS/Fargate, RDS PostgreSQL, SQS, Textract, EventBridge, SES — nhưng tùy biến vai trò từng dịch vụ theo nghiệp vụ đặc thù của ngành hàng không.

#### Sơ đồ kiến trúc tổng quan

![Kiến trúc Airport Management System](/images/5-Workshop/airport-architecture-overview.png)

*Hình 2.1 — Sơ đồ kiến trúc tổng quan của Airport Management System.*

#### Nội dung

- [Danh sách dịch vụ AWS sử dụng](5.3.1-services/)
- [Luồng dữ liệu chi tiết](5.3.2-dataflow/)
