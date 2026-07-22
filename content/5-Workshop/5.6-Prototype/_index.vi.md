---
title : "Chạy thử prototype"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### Từ kiến trúc đến mã nguồn chạy được

Các phần trước mô tả thiết kế. Phần này chạy thử thiết kế đó.

Mã nguồn được nộp kèm báo cáo này dưới dạng một gói riêng,
**`TranQuocKhanh-Project`**. Đây là bản cài đặt hoạt động được của pipeline OCR
và API nghiệp vụ, được xây dựng để chạy hoàn toàn trên máy cá nhân — **không cần
tài khoản AWS và không tốn chi phí**. Amazon S3 và Amazon SQS được giả lập bằng
[LocalStack](https://localstack.cloud/), PostgreSQL chạy trong container thay cho
RDS, và hai hàm Lambda chạy dưới dạng tiến trình Python thông thường.

Hãy giải nén `TranQuocKhanh-Project.zip` để thực hành theo; mọi đường dẫn bên
dưới đều tính từ thư mục `TranQuocKhanh-Project/` sau khi giải nén.

#### Ánh xạ giữa mã nguồn và kiến trúc

| Prototype | Kiến trúc triển khai thật |
|---|---|
| nginx trong `edge/` | Route 53 → CloudFront → WAF → ALB |
| Container `api/` | ECS/Fargate task đứng sau ALB |
| `worker/src/handlers.py` | Hai hàm Lambda với SQS event source mapping |
| `worker/src/scheduled.py` | Một hàm Lambda chạy theo lịch cron của EventBridge |
| `worker/src/poller.py` | Cơ chế polling SQS của chính dịch vụ Lambda — không có tương đương trên AWS |
| Chế độ dev trong `shared/auth.py` | Amazon Cognito user pool (RS256 qua JWKS) |
| `shared/idempotency.py` | Amazon DynamoDB với conditional write |
| `shared/observability.py` | Amazon CloudWatch metrics và alarm |
| Quét cục bộ trong `shared/pii.py` | Amazon Macie classification job |
| `shared/parameters.py` | SSM Parameter Store + Secrets Manager (KMS) |
| LocalStack | S3, SQS, DynamoDB, CloudWatch, EventBridge, SSM, Secrets Manager, KMS |
| Provider `mock` trong `ocr.py` | Amazon Textract |
| Provider `log` trong `notifier.py` | Amazon SES |
| Container `postgres` | Amazon RDS for PostgreSQL, Multi-AZ |
| Mạng `private` trong compose | Private subnet không có đường ra Internet |
| `endpoint_url` trong `shared/aws.py` | VPC Endpoint |
| `buildspec.yml` / `appspec.yml` | CodeBuild → ECR → CodeDeploy (blue/green) |

{{% notice note %}}
Các giai đoạn của pipeline được viết dưới dạng Lambda handler — nhận `event`
theo đúng định dạng SQS và trả về partial-batch response — nên có thể đưa
nguyên mã nguồn này lên Lambda thật. Khác biệt duy nhất giữa môi trường local
và môi trường triển khai là `endpoint_url` trong `shared/aws.py`.
{{% /notice %}}

#### Xác thực

Ở môi trường local, `AUTH_MODE=dev` xác thực token HS256 ký bằng khoá bí mật
dùng chung, vì bản community của LocalStack không có Cognito; trên AWS,
`AUTH_MODE=cognito` xác thực token RS256 với JWKS endpoint của user pool. Cả hai
đều kiểm tra thật chữ ký, issuer và hạn sử dụng, và ứng dụng chỉ nhìn thấy
claim `sub` dưới dạng `owner_id`.

| Endpoint | Quyền truy cập |
|---|---|
| `/health`, `/docs` | Công khai |
| `GET /flights`, `GET /airports` | Công khai — đúng lịch bay hiển thị trên bảng thông báo |
| `GET /documents`, `GET /notifications` | Cần token; giới hạn theo người gọi |
| `GET /boarding-passes`, `GET /passengers` | Cần token; nhân viên xem tất cả, hành khách chỉ xem của mình |
| `PATCH /flights/{id}/status`, `POST /passengers` | Cần token **và** thuộc nhóm `staff` |

Lịch chuyến bay được chủ động để công khai — hành khách tra cứu giờ khởi hành
không cần phải có tài khoản. Mọi dữ liệu cá nhân (tên, số ghế, số hộ chiếu, mã
vạch) đều yêu cầu xác thực.

{{% notice warning %}}
`SERVICES.md` trong gói dự án là danh sách đối chiếu chính thức. Tài liệu này ghi
rõ dịch vụ nào đã được **kiểm chứng bằng test**, dịch vụ nào mới ở mức **cấu
hình** — đã viết và kiểm tra cú pháp nhưng chưa từng chạy — và những dịch vụ nào
trong worklog đã được **chủ động loại trừ**, kèm lý do cho từng trường hợp.
{{% /notice %}}

#### Nội dung

- [Yêu cầu và khởi chạy](5.6.1-run/)
- [Đi qua pipeline OCR](5.6.2-pipeline/)
- [Dọn dẹp](5.6.3-cleanup/)
