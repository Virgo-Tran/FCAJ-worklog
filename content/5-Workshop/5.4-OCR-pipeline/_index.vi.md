---
title : "Pipeline xử lý tài liệu OCR"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

Đây là quy trình nghiệp vụ đặc thù quan trọng nhất của hệ thống: tự động trích xuất dữ liệu từ boarding pass và thẻ hành lý mà nhân viên hoặc hành khách tải lên, giảm thao tác nhập liệu thủ công.

![Pipeline xử lý tài liệu OCR](/images/5-Workshop/ocr-pipeline.png)

*Hình 2.2 — Pipeline xử lý tài liệu OCR: Textract → SQS OCR Results → Lambda Process OCR Result → RDS → SES.*

#### Các bước trong pipeline

1. Nhân viên sân bay yêu cầu Presigned URL; **Lambda Gen Presigned URL** sinh URL này.
2. Tài liệu được tải trực tiếp lên bucket **S3 Airport Docs**.
3. S3 phát sự kiện `Object Created` vào **SQS Document Queue**.
4. **Lambda Submit OCR Job** đọc hàng đợi và gọi **Amazon Textract** qua VPC Endpoint.
5. Textract hoàn tất OCR và đẩy kết quả vào **SQS OCR Results**.
6. **Lambda Process OCR Result** đọc hàng đợi, lưu dữ liệu trích xuất vào **RDS PostgreSQL** và gửi thông báo qua **SES**.

#### Các trường dữ liệu trích xuất từ boarding pass

Textract trích xuất các trường dữ liệu quan trọng sau:

+ Tên hành khách
+ Số hiệu chuyến bay
+ Giờ khởi hành / đến
+ Số ghế
+ Cổng
+ Giờ lên máy bay
+ Số hộ chiếu
+ Dữ liệu mã vạch

#### Xử lý lỗi

{{% notice warning %}}
Các message xử lý thất bại (quá số lần thử) được chuyển vào **Dead Letter Queue** và **CloudWatch** sẽ cảnh báo để đội vận hành xử lý.
{{% /notice %}}
