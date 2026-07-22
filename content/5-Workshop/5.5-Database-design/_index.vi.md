---
title : "Thiết kế cơ sở dữ liệu"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

Lược đồ cơ sở dữ liệu RDS PostgreSQL gồm **6 bảng chính**, được chuẩn hoá theo mô hình quan hệ.

#### Danh sách bảng

| Bảng | Khoá chính | Trường chính khác | Quan hệ / Ghi chú |
|---|---|---|---|
| **airports** | `airport_id` (PK) | `name`, `code` (CHAR 3), `city`, `country`, `timezone`, `created_at`, `updated_at` | Bảng gốc — 1 sân bay có nhiều chuyến bay khởi hành/đến (1:N với `flights`). |
| **flights** | `flight_id` (PK) | `departure_airport_id` (FK), `arrival_airport_id` (FK), `airline`, `flight_number`, `departure_time`, `arrival_time`, `status` (ENUM), `gate`, `terminal`, `aircraft_type`, `created_at` | Một chuyến bay có nhiều boarding pass và có thể sinh ra nhiều notification (1:N). |
| **passengers** | `passenger_id` (PK) | `owner_id` (Cognito FK), `full_name`, `passport_number`, `nationality`, `date_of_birth`, `email`, `phone`, `created_at` | Một hành khách có thể có nhiều boarding pass (1:N), liên kết Cognito qua `owner_id`. |
| **documents** | `doc_id` (PK) | `owner_id` (Cognito FK), `s3_key`, `s3_bucket`, `doc_type` (ENUM), `ocr_status` (ENUM), `ocr_text`, `ocr_confidence`, `extracted_data` (JSONB), `file_size_bytes`, `uploaded_at`, `processed_at` | Lưu metadata + kết quả OCR của từng tài liệu tải lên; liên kết tới `boarding_passes` qua `document_id`. |
| **boarding_passes** | `pass_id` (PK) | `flight_id` (FK), `passenger_id` (FK), `document_id` (FK), `seat_number`, `boarding_group`, `gate`, `boarding_time`, `status` (ENUM), `barcode_data`, `created_at` | Bảng trung tâm nối `flights` – `passengers` – `documents`. |
| **notifications** | `notif_id` (PK) | `user_id` (Cognito FK), `flight_id` (FK), `type` (ENUM), `subject`, `message`, `channel` (ENUM), `status` (ENUM), `sent_at`, `created_at` | Lưu lịch sử thông báo gửi qua SES cho từng chuyến bay/người dùng. |

#### Quan hệ giữa các bảng

```text
airports    (1) ──< (N)  flights
flights     (1) ──< (N)  boarding_passes
passengers  (1) ──< (N)  boarding_passes
documents   (1) ──< (N)  boarding_passes   (qua document_id)
flights     (1) ──< (N)  notifications
```

Bảng `boarding_passes` đóng vai trò bảng trung tâm liên kết chuyến bay, hành khách và tài liệu đã tải lên.

#### Các trường ENUM quan trọng

| Trường | Giá trị hợp lệ |
|---|---|
| `flight.status` | `scheduled`, `boarding`, `departed`, `arrived`, `delayed`, `cancelled` |
| `boarding_pass.status` | `issued`, `checked_in`, `boarded`, `no_show` |
| `document.doc_type` | `boarding_pass`, `luggage_tag`, `passport`, `visa`, `id_card` |
| `document.ocr_status` | `pending`, `processing`, `completed`, `failed` |
| `notification.type` | `delay`, `gate_change`, `boarding`, `cancellation`, `monthly` |
