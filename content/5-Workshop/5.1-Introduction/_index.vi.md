---
title : "Giới thiệu"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Bài toán

Mỗi ngày sân bay phải xử lý một lượng lớn tài liệu dạng giấy và hình ảnh: thẻ lên máy bay (boarding pass), thẻ hành lý (luggage tag), hộ chiếu. Hiện nay phần lớn dữ liệu này được nhân viên nhập tay vào hệ thống nội bộ — chậm, dễ sai sót và không thể mở rộng trong giờ cao điểm.

**Hệ thống Quản lý Thông tin Sân bay** giải quyết bài toán này, phục vụ hai nhóm người dùng chính:

+ **Hành khách** — tra cứu thông tin chuyến bay, tải lên thẻ lên máy bay và nhận thông báo.
+ **Nhân viên sân bay** — quản lý chuyến bay, hồ sơ hành khách và xử lý tài liệu được tải lên.

#### Chức năng chính

Hệ thống cung cấp:

1. **Xác thực người dùng** — hành khách và nhân viên đăng nhập và nhận JWT.
2. **Quản lý chuyến bay và hành khách** — quản lý đầy đủ thông tin sân bay, chuyến bay và hồ sơ hành khách.
3. **Tự động trích xuất dữ liệu bằng OCR** — tải lên thẻ lên máy bay hoặc thẻ hành lý, hệ thống tự động trích xuất tên hành khách, số hiệu chuyến bay, giờ khởi hành/đến, số ghế, cổng, số hộ chiếu và dữ liệu mã vạch mà không cần nhập liệu thủ công.
4. **Thông báo tự động** — gửi email khi chuyến bay trễ giờ, đổi cổng, nhắc lên máy bay, huỷ chuyến, cùng thông báo tổng hợp hàng tháng.

#### Cách tiếp cận

Dự án được xây dựng dựa trên **kiến trúc mẫu AWS V2** trình bày ở [phần tiếp theo](../5.2-Reference-architecture/). Dự án áp dụng đúng công nghệ nền tảng — ECS/Fargate, RDS PostgreSQL, SQS, Textract, EventBridge, SES — nhưng tùy biến vai trò từng dịch vụ theo nghiệp vụ đặc thù của ngành hàng không.

{{% notice note %}}
Workshop này được chia thành hai phần, phản ánh đúng cách hệ thống được thiết kế: trước tiên là **kiến trúc tham khảo tổng quát** thiết lập các mẫu thiết kế, sau đó là **hệ thống sân bay cụ thể** áp dụng các mẫu đó.
{{% /notice %}}
