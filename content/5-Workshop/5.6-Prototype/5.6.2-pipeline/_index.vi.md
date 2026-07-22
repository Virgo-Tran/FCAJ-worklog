---
title : "Đi qua pipeline OCR"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2 </b> "
---

#### Chạy demo

Khi hệ thống đã chạy, mở một terminal thứ hai:

```bash
cd TranQuocKhanh-Project
python scripts/demo.py
```

Script chỉ dùng thư viện chuẩn của Python. Script đi qua toàn bộ pipeline và in
ra từng giai đoạn: xin Presigned URL, tải lên một boarding pass được sinh tự động,
chờ OCR, sau đó hiển thị các trường đã trích xuất, boarding pass được tạo từ
chúng và các thông báo được sinh ra.

#### Diễn biến từng bước

**Bước 1 — Presigned URL.** API thêm một bản ghi `documents` với
`ocr_status = 'pending'` và trả về Presigned URL dạng `PUT`. Tệp không hề đi
qua API:

```bash
curl -X POST "http://localhost:8000/documents/upload-url?owner_id=demo-user-0001" \
  -H "Content-Type: application/json" \
  -d '{"filename":"boarding-pass.png","doc_type":"boarding_pass"}'
```

**Bước 2 — Tải lên.** Client `PUT` tệp trực tiếp lên S3. Trên AWS, luồng này đi
qua S3 Gateway Endpoint thay vì qua Internet.

**Bước 3 — Sự kiện.** S3 phát sự kiện `ObjectCreated`, message rơi vào
**hàng đợi xử lý tài liệu**.

**Bước 4 — OCR.** `submit_job_handler` nhận message, tải object về, chạy OCR và
đẩy kết quả vào **hàng đợi kết quả OCR**. Trên AWS đây là một hàm Lambda gọi
Amazon Textract qua VPC Endpoint.

**Bước 5 — Lưu trữ.** `process_result_handler` đọc kết quả, ghi `ocr_text`,
`ocr_confidence` và `extracted_data` (JSONB) vào bản ghi tài liệu, đặt
`ocr_status = 'completed'`, rồi đối chiếu số hiệu chuyến bay và tên hành khách
đã trích xuất với bản ghi trong `boarding_passes`.

**Bước 6 — Thông báo.** Một bản ghi `notifications` được tạo và gửi đi. Với
`NOTIFIER=log`, nội dung email được in ra log của worker:

```bash
docker compose logs worker | grep -A5 EMAIL
```

#### Kiểm tra kết quả

```bash
curl "http://localhost:8000/documents?owner_id=demo-user-0001" | python -m json.tool
curl "http://localhost:8000/boarding-passes" | python -m json.tool
curl "http://localhost:8000/notifications?user_id=demo-user-0001" | python -m json.tool
```

Hoặc truy vấn trực tiếp cơ sở dữ liệu:

```sql
SELECT doc_id, ocr_status, ocr_confidence, extracted_data
FROM documents ORDER BY doc_id DESC LIMIT 1;
```

#### Thử luồng thông báo

Thay đổi trạng thái chuyến bay sẽ tạo thông báo cho mọi hành khách có boarding
pass trên chuyến đó:

```bash
curl -X PATCH http://localhost:8000/flights/1/status \
  -H "Content-Type: application/json" \
  -d '{"status":"delayed","notify":true}'
```

Nếu truyền `gate` khác thay vì trạng thái, hệ thống sẽ sinh thông báo loại
`gate_change`.

#### Quan sát Dead Letter Queue hoạt động

Tải lên một object mà pipeline không xử lý được, và quan sát nó được thử lại ba
lần trước khi rơi vào DLQ — đúng luồng xử lý lỗi ở phần 5.4:

```bash
docker compose exec localstack \
  awslocal s3 cp /etc/hostname s3://airport-documents/demo-user-0001/boarding_pass/broken.png

# quan sát ba lần xử lý thất bại
docker compose logs -f worker

# sau khoảng một phút, message nằm trong DLQ
docker compose exec localstack \
  awslocal sqs receive-message --queue-url \
  "$(docker compose exec -T localstack awslocal sqs get-queue-url \
     --queue-name airport-document-dlq --output text --query QueueUrl)"
```

{{% notice tip %}}
Trên AWS, chính tại thời điểm này một CloudWatch alarm đặt trên chỉ số
`ApproximateNumberOfMessagesVisible` của DLQ sẽ cảnh báo cho đội vận hành.
{{% /notice %}}
