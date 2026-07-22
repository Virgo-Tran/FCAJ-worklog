---
title : "Yêu cầu và khởi chạy"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1 </b> "
---

#### Yêu cầu chuẩn bị

+ **Docker Desktop** (hoặc bất kỳ bản Docker nào có Compose v2). Không cần cài gì
  thêm — Python, PostgreSQL và AWS CLI đều chạy bên trong container.
+ Khoảng 2 GB dung lượng trống cho các image.
+ Các cổng **8000** (API), **5432** (PostgreSQL) và **4566** (LocalStack) còn trống.

{{% notice note %}}
Không cần tài khoản AWS và không phát sinh chi phí. Amazon S3 và Amazon SQS được
giả lập bằng LocalStack; Textract và SES được thay bằng provider chạy cục bộ.
{{% /notice %}}

#### Khởi chạy hệ thống

```bash
cd TranQuocKhanh-Project
docker compose up --build
```

Lần chạy đầu tiên mất vài phút. Compose khởi động bốn service theo thứ tự:

1. **postgres** — tạo schema và dữ liệu mẫu từ `db/*.sql` trong lần khởi động đầu.
2. **localstack** — tạo S3 bucket, ba hàng đợi, redrive policy cho DLQ và
   cấu hình thông báo sự kiện S3 → SQS.
3. **api** — chờ hai service trên báo healthy, sau đó phục vụ ở cổng 8000.
4. **worker** — bắt đầu polling hàng đợi tài liệu và hàng đợi kết quả OCR.

Chờ đến khi API báo healthy:

```bash
curl http://localhost:8000/health
# {"status":"ok","database":"ok"}
```

#### Khám phá API

Mở <http://localhost:8000/docs> để xem giao diện Swagger được sinh tự động.
Dữ liệu mẫu có sẵn bốn sân bay, ba chuyến bay và ba hành khách:

```bash
curl http://localhost:8000/flights
curl http://localhost:8000/airports
```

#### Các lệnh hữu ích

| Mục đích | Lệnh |
|---|---|
| Xem log worker | `docker compose logs -f worker` |
| Vào shell cơ sở dữ liệu | `docker compose exec postgres psql -U airport -d airport` |
| Liệt kê hàng đợi | `docker compose exec localstack awslocal sqs list-queues` |
| Liệt kê tài liệu đã tải lên | `docker compose exec localstack awslocal s3 ls s3://airport-documents --recursive` |
