---
title: "Blog 2: AWS Compute Decision Framework"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Lambda, ECS Fargate, EKS hay EC2 — Lựa chọn nền tảng Compute nào cho phù hợp?

> **Chủ đề:** *AWS Compute Decision Framework*  
> **Tác giả:** Trần Quốc Khánh  
> **Cộng đồng đăng tải:** [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj)  

*Khung hướng dẫn ra quyết định thực tiễn dành cho Kiến trúc sư và Lập trình viên Backend để chọn đúng nền tảng tính toán cho từng loại ứng dụng.*

---

Nếu bạn từng tham gia một buổi review kiến trúc AWS, nhiều khả năng bạn đã gặp câu hỏi quen thuộc: *"Ứng dụng này nên chạy trên Lambda, ECS, EKS hay EC2?"* Đây không phải là một câu hỏi lý thuyết — nó ảnh hưởng trực tiếp đến chi phí vận hành, tốc độ triển khai và khối lượng công việc quản lý của đội ngũ kỹ thuật.

AWS cung cấp ít nhất 6 dịch vụ tính toán cốt lõi, và mỗi dịch vụ được tối ưu hóa cho một loại bài toán khác nhau. Bài viết này trình bày một khung ra quyết định thực tiễn dựa trên đặc điểm lưu lượng (traffic), yêu cầu độ trễ (latency), năng lực vận hành của team và chi phí.

---

### 1. Bài toán đánh đổi cốt lõi: Kiểm soát vs. Chi phí vận hành

Mọi tùy chọn Compute trên AWS đều nằm trên cùng một đường cong đánh đổi: **Quyền kiểm soát hạ tầng càng cao thì chi phí vận hành càng lớn**.

* **Amazon EC2:** Cho bạn toàn quyền kiểm soát máy ảo (VM), hệ điều hành, runtime, bản vá bảo mật và auto scaling. Đổi lại, team bạn phải tự quản lý toàn bộ các công việc vận hành đó.
* **AWS Fargate (trên Amazon ECS):** Loại bỏ gánh nặng quản lý máy chủ. Bạn chỉ làm việc ở cấp độ container (image và task definition), AWS quản lý OS và cluster bên dưới.
* **Amazon EKS:** Môi trường Kubernetes tiêu chuẩn, đầy đủ tính năng với Control Plane được AWS quản lý. Rất phù hợp cho kiến trúc microservices lớn, nhưng đòi hỏi năng lực vận hành cao.
* **AWS Lambda:** Mô hình Serverless hoàn toàn. Bạn chỉ tập trung viết code hàm, AWS lo liệu mọi thứ từ cấp phát đến tự động mở rộng theo lưu lượng.
* **AWS App Runner:** Dịch vụ container đơn giản hóa tối đa cho web app chạy liên tục, gần như không cần cấu hình hạ tầng.
* **AWS Batch:** Được thiết kế riêng cho các công việc tính toán theo đợt (batch job). Tự động cấp phát và giải phóng tài nguyên dựa trên hàng đợi công việc.

---

### 2. Phân tích chi tiết các dịch vụ chính

#### AWS Lambda — Tối ưu cho kiến trúc hướng sự kiện (Event-Driven)
Phù hợp nhất để xử lý file tải lên S3, nhận tin nhắn từ SQS, xử lý request từ API Gateway hoặc phản hồi Stream từ DynamoDB. Mô hình tính phí theo lượt dùng (pay-per-use) giúp Lambda đạt hiệu quả chi phí tuyệt đối với lưu lượng thất thường.

* **Giới hạn cần nhớ:** Thời gian chạy tối đa 15 phút; có độ trễ Cold Start; giới hạn 1,000 concurrent executions mặc định; không hỗ trợ GPU.

#### Amazon ECS Fargate — Điểm cân bằng giữa Lambda và EKS
Fargate phù hợp khi ứng dụng cần chạy liên tục 24/7, duy trì in-memory state, đòi hỏi custom runtime mà Lambda không hỗ trợ, hoặc khi traffic cao và ổn định (trên 50-100 triệu request/tháng, chi phí Fargate bắt đầu rẻ hơn Lambda).

#### Amazon EKS — Sức mạnh cho quy mô lớn
EKS chỉ nên là ưu tiên hàng đầu trong 3 trường hợp: Team đã có kinh nghiệm Kubernetes production; hệ thống có yêu cầu bắt buộc về tính đa đám mây (multi-cloud); hoặc cần tích hợp sâu với hệ sinh thái Cloud-Native (Helm, Istio, Linkerd, GitOps).

#### Amazon EC2 — Kiểm soát tuyệt đối cho Workload đặc thù
Phù hợp cho các bài toán huấn luyện AI/ML cần GPU cứng (p4d, g5, p5), ứng dụng giữ kết nối lâu dài (WebSocket, Game server), hệ thống legacy hoặc lưu lượng cực lớn sử dụng Reserved Instances / Savings Plans.

---

### 3. Bảng tra cứu nhanh theo kịch bản thực tế

| Kịch bản ứng dụng | Dịch vụ Compute khuyến nghị | Lý do kỹ thuật |
| :--- | :--- | :--- |
| **Event webhook receiver** (S3, GitHub, Stripe) | **AWS Lambda** | Tác vụ ngắn, hướng sự kiện; chi phí gần như bằng 0 khi nhàn rỗi. |
| **REST API traffic thấp hoặc thất thường** | **AWS Lambda** hoặc **AWS App Runner** | Khả năng scale về 0; App Runner phù hợp web app chạy liên tục đơn giản. |
| **REST API traffic cao và ổn định** | **Amazon ECS Fargate** hoặc **Amazon EC2** | Chi phí theo request của Lambda không còn tối ưu ở quy mô lớn. |
| **API yêu cầu độ trễ sub-100ms p99** | **Amazon ECS Fargate** / **AWS App Runner** | Tránh sự biến động độ trễ do Cold Start của Lambda. |
| **Scheduled job chạy dưới 15 phút** | **AWS Lambda + Amazon EventBridge** | Không phát sinh chi phí hạ tầng đứng chờ giữa các lần chạy. |
| **Scheduled job chạy trên 15 phút** | **AWS Batch** hoặc **Amazon ECS Task** | Vượt qua giới hạn thời gian 15 phút của Lambda. |
| **Stateful WebSocket / Real-time connection** | **Amazon ECS Fargate** hoặc **Amazon EC2** | Lambda không được thiết kế để giữ kết nối liên tục. |
| **Huấn luyện ML cần tài nguyên GPU** | **Amazon EC2 (GPU)** hoặc **AWS Batch** | Lambda và Fargate không hỗ trợ tăng tốc phần cứng GPU. |
| **Hệ thống Microservices (5-20 services)** | **Amazon ECS Fargate** | Đạt chuẩn Production mà không có độ phức tạp của cụm K8s. |
| **Hệ thống Microservices lớn, đa đám mây** | **Amazon EKS** | Giữ tính nhất quán của các công cụ và manifest chuẩn Kubernetes. |

---

### 4. Tổng chi phí sở hữu TCO: Nhìn xa hơn hóa đơn Compute

Khi tính toán TCO cho một hệ thống (ví dụ: 100 triệu request/tháng), ngoài hóa đơn Compute trực tiếp, bạn cần tính đến các chi phí ẩn và vận hành:
* **Chi phí Nhân sự (Staffing costs):** Đặc biệt là thời gian kỹ sư dành cho việc quản lý cụm EKS.
* **Hạ tầng bổ trợ (Supporting infrastructure):** Phí cố định ALB, lưu trữ container image trên ECR, lưu trữ log trên CloudWatch.
* **Chi phí Data Transfer Out:** Chi phí thầm lặng nhưng rất đáng kể trên AWS.

---

### 5. Bộ 6 câu hỏi ra quyết định nhanh (Decision Framework)

1. *Workload có theo sự kiện và hoàn thành dưới 15 phút không?* **Có → AWS Lambda.**
2. *Lưu lượng ổn định hay thất thường?* **Thất thường → AWS Lambda.** **Ổn định → Lambda Managed Instances, ECS Fargate, hoặc EKS.**
3. *Ứng dụng có cần GPU, truy cập sâu kernel OS, hay lưu trữ local không?* **Có → Amazon EC2.**
4. *Hệ thống có bắt buộc dùng công cụ Kubernetes hoặc cần linh hoạt đa đám mây không?* **Có → Amazon EKS.** **Không → Amazon ECS Fargate.**
5. *Tác vụ chạy lâu hơn 15 phút và cần lập lịch định kỳ?* **Có → AWS Batch.**
6. *Team có chuyên môn sâu về K8s và hệ thống có dưới 10 services không?* **Ít service, không có kinh nghiệm K8s → Chọn Amazon ECS Fargate.**

---

### 6. Kết luận: Kiến trúc Production tối ưu thường là Hybrid

Trong môi trường thực tế, rất ít hệ thống lớn chỉ chạy trên một dịch vụ compute duy nhất. Mô hình kết hợp (Hybrid Compute Model) phổ biến: **API Gateway + Lambda** tiếp nhận webhook, **ECS Fargate** chạy các core service liên tục, và **AWS Batch** xử lý tính toán đợt nặng cuối ngày.

