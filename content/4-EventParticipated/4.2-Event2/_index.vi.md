---
title: "FCAJ Community Day (Tháng 6)"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# Báo cáo tổng kết: FCAJ Community Day - June 2026 Edition

* **Thời gian:** 08:30 – 12:00, Thứ Bảy, ngày 20/06/2026  
* **Địa điểm:** Văn phòng AWS Việt Nam & Trực tuyến  
* **Ban tổ chức:** AWS Study Group / FCAJ Community Team  
* **Video lưu trữ sự kiện:** [YouTube - FCAJ Community Day June 2026](https://www.youtube.com/watch?v=G8-WlI7f6dE)  
* **Vai trò:** Người tham dự (Thực tập sinh FCAJ)  

---

### Mục tiêu sự kiện

* Tiếp nối chuỗi sự kiện sinh hoạt chuyên môn định kỳ thuộc chương trình **First Cloud AI Journey (FCAJ)** tại AWS Việt Nam.
* Đột phá chuyên sâu vào các chuyên đề hạ tầng container nâng cao (Amazon EKS / ECS), giải pháp tối ưu hóa chi phí đám mây (FinOps), và mô hình Serverless thế hệ mới.
* Giao lưu, chia sẻ kinh nghiệm xử lý sự cố (Troubleshooting) và lắng nghe các phản hồi thực tế từ đội ngũ Mentor và các kỹ sư Cloud trong cộng đồng.

---

### Diễn giả & Chủ đề chia sẻ

1. **AWS Study Group Mentors** — *Advanced EKS Operations & Cost Right-Sizing in Practice*
2. **FCAJ Technical Leads** — *Building Production-Ready Event-Driven Pipelines with AWS Lambda & SQS*
3. **Cloud Security Specialists** — *Identity & Access Governance: Least Privilege with AWS IAM & KMS Encryption*

---

### Nội dung nổi bật

#### 1. Tối ưu hóa chi phí cụm EKS & Vận hành Container ở quy mô Production
* Đi sâu vào bài toán "Right-sizing" tài nguyên Pod thực tế bằng CloudWatch Container Insights và giải pháp tự động cấp phát Node động với Karpenter.
* Hướng dẫn chiến lược kết hợp Spot Instances, Graviton ARM CPU và loại bỏ chi phí NAT Gateway bằng VPC Endpoints để giảm thiểu từ 40-60% hóa đơn hạ tầng.
* Phân tích các mô hình mua Compute (Savings Plans vs Reserved Instances) phù hợp cho từng giai đoạn phát triển của doanh nghiệp.

#### 2. Xây dựng Pipeline Serverless hướng sự kiện đạt chuẩn Production
* Hướng dẫn thực hành kiến trúc Event-Driven nâng cao kết hợp giữa Presigned URL S3, hàng đợi SQS, Lambda và dịch vụ trí tuệ nhân tạo (Amazon Textract).
* Xây dựng cơ chế xử lý lỗi bền vững với Dead Letter Queue (DLQ), cảnh báo CloudWatch Alarm tự động và truy vết hệ thống qua AWS X-Ray.
* Giới thiệu các tính năng Serverless Workflow thế hệ mới giúp kiểm soát trạng thái dữ liệu một cách hiệu quả.

#### 3. Quản trị danh tính & Bảo mật dữ liệu đám mây (IAM & KMS)
* Nguyên tắc bảo mật đặc quyền tối thiểu (Least Privilege) trong thiết lập chính sách IAM cho dịch vụ và ứng dụng.
* Mô hình mã hóa dữ liệu tại chỗ (Encryption at Rest) và mã hóa trên đường truyền (Encryption in Transit) sử dụng AWS KMS.
* Giám sát mối đe dọa tự động và đánh giá tuân thủ bảo mật với GuardDuty, Security Hub và AWS Macie.

---

### Bài học rút ra

* **Tư duy tối ưu hóa chi phí thầm lặng (FinOps):** Nhận thức rõ ràng rằng việc thiết kế kiến trúc chuẩn không chỉ là chạy được mà còn phải tối ưu hóa tài nguyên và chi phí vận hành ngay từ đầu.
* **Kỹ năng thiết kế hệ thống chịu lỗi (Resilient Architecture):** Thấy được tầm quan trọng của các tầng xử lý lỗi (DLQ, Alarms, Retry strategies) trong các ứng dụng thực tế.
* **Nâng cao nhận thức bảo mật:** Nắm vững cách xây dựng các chính sách IAM chặt chẽ và áp dụng mã hóa bảo vệ tài sản dữ liệu trên Cloud.

---

### Áp dụng vào công việc & dự án thực tập

* Áp dụng trực tiếp các phương pháp chuẩn hóa VPC Endpoint, IAM Policy và xử lý lỗi DLQ vào dự án **Hệ thống Quản lý Thông tin Sân bay (AIMS)**.
* Sử dụng các nguyên tắc tối ưu hóa chi phí để rà soát và đánh giá cấu hình hạ tầng cho bài báo cáo thực tập cuối kỳ.
* Đóng góp bài viết tổng hợp chuyên sâu lên cộng đồng AWS Study Group để chia sẻ kiến thức với các bạn thực tập sinh khác.

---

### Trải nghiệm sự kiện

Phiên sinh hoạt tháng 6 mang lại nhiều kiến thức chuyên môn sâu sắc và tính thực chiến cao. Sự tương tác sôi nổi giữa các diễn giả và người tham dự qua phần Q&A đã giúp tôi giải đáp trực tiếp nhiều vướng mắc kỹ thuật gặp phải trong quá trình làm dự án thực tập.
