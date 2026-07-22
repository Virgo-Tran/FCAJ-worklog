---
title : "Database design"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

The RDS PostgreSQL schema consists of **6 main tables**, normalized following the relational model.

#### Tables

| Table | Primary key | Other key fields | Relationships / notes |
|---|---|---|---|
| **airports** | `airport_id` (PK) | `name`, `code` (CHAR 3), `city`, `country`, `timezone`, `created_at`, `updated_at` | Root table — one airport has many departing and arriving flights (1:N with `flights`). |
| **flights** | `flight_id` (PK) | `departure_airport_id` (FK), `arrival_airport_id` (FK), `airline`, `flight_number`, `departure_time`, `arrival_time`, `status` (ENUM), `gate`, `terminal`, `aircraft_type`, `created_at` | One flight has many boarding passes and can generate many notifications (1:N). |
| **passengers** | `passenger_id` (PK) | `owner_id` (Cognito FK), `full_name`, `passport_number`, `nationality`, `date_of_birth`, `email`, `phone`, `created_at` | One passenger can hold many boarding passes (1:N); linked to Cognito via `owner_id`. |
| **documents** | `doc_id` (PK) | `owner_id` (Cognito FK), `s3_key`, `s3_bucket`, `doc_type` (ENUM), `ocr_status` (ENUM), `ocr_text`, `ocr_confidence`, `extracted_data` (JSONB), `file_size_bytes`, `uploaded_at`, `processed_at` | Stores metadata plus the OCR result for each uploaded document; links to `boarding_passes` via `document_id`. |
| **boarding_passes** | `pass_id` (PK) | `flight_id` (FK), `passenger_id` (FK), `document_id` (FK), `seat_number`, `boarding_group`, `gate`, `boarding_time`, `status` (ENUM), `barcode_data`, `created_at` | The central table joining flights, passengers, and documents. |
| **notifications** | `notif_id` (PK) | `user_id` (Cognito FK), `flight_id` (FK), `type` (ENUM), `subject`, `message`, `channel` (ENUM), `status` (ENUM), `sent_at`, `created_at` | History of notifications sent via SES for each flight and user. |

#### Relationships

```text
airports    (1) ──< (N)  flights
flights     (1) ──< (N)  boarding_passes
passengers  (1) ──< (N)  boarding_passes
documents   (1) ──< (N)  boarding_passes   (via document_id)
flights     (1) ──< (N)  notifications
```

`boarding_passes` acts as the central table connecting flights, passengers, and uploaded documents.

#### Key ENUM values

| Field | Allowed values |
|---|---|
| `flight.status` | `scheduled`, `boarding`, `departed`, `arrived`, `delayed`, `cancelled` |
| `boarding_pass.status` | `issued`, `checked_in`, `boarded`, `no_show` |
| `document.doc_type` | `boarding_pass`, `luggage_tag`, `passport`, `visa`, `id_card` |
| `document.ocr_status` | `pending`, `processing`, `completed`, `failed` |
| `notification.type` | `delay`, `gate_change`, `boarding`, `cancellation`, `monthly` |
