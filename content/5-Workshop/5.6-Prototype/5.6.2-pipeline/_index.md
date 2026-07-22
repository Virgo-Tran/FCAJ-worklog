---
title : "Walking the OCR pipeline"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2 </b> "
---

#### Run the demo

With the stack up, in a second terminal:

```bash
cd TranQuocKhanh-Project
python scripts/demo.py
```

The script uses only the Python standard library. It walks the whole pipeline
and prints each stage: requests a presigned URL, uploads a generated boarding
pass, waits for OCR, then shows the extracted fields, the boarding pass created
from them, and the notifications produced.

#### What happens at each step

**Step 1 — presigned URL.** The API inserts a `documents` row with
`ocr_status = 'pending'` and returns a presigned `PUT` URL. The file never
passes through the API:

```bash
curl -X POST "http://localhost:8000/documents/upload-url?owner_id=demo-user-0001" \
  -H "Content-Type: application/json" \
  -d '{"filename":"boarding-pass.png","doc_type":"boarding_pass"}'
```

**Step 2 — upload.** The client `PUT`s the file straight to S3. In AWS this
traffic goes through the S3 Gateway Endpoint rather than the Internet.

**Step 3 — the event.** S3 emits `ObjectCreated`, which lands on the
**document processing queue**.

**Step 4 — OCR.** `submit_job_handler` picks the message up, downloads the
object, runs OCR, and publishes the result to the **OCR results queue**. In AWS
this is a Lambda calling Amazon Textract through a VPC Endpoint.

**Step 5 — persistence.** `process_result_handler` reads the result, writes
`ocr_text`, `ocr_confidence` and `extracted_data` (JSONB) onto the document,
sets `ocr_status = 'completed'`, then reconciles the extracted flight number
and passenger name against a `boarding_passes` row.

**Step 6 — notification.** A `notifications` row is written and delivered. With
`NOTIFIER=log` the email body is printed to the worker log:

```bash
docker compose logs worker | grep -A5 EMAIL
```

#### Inspect the result

```bash
curl "http://localhost:8000/documents?owner_id=demo-user-0001" | python -m json.tool
curl "http://localhost:8000/boarding-passes" | python -m json.tool
curl "http://localhost:8000/notifications?user_id=demo-user-0001" | python -m json.tool
```

Or straight from the database:

```sql
SELECT doc_id, ocr_status, ocr_confidence, extracted_data
FROM documents ORDER BY doc_id DESC LIMIT 1;
```

#### Exercise the notification path

Changing a flight's status queues a notification to every passenger holding a
boarding pass on it:

```bash
curl -X PATCH http://localhost:8000/flights/1/status \
  -H "Content-Type: application/json" \
  -d '{"status":"delayed","notify":true}'
```

Setting a different `gate` instead produces a `gate_change` notification.

#### See the Dead Letter Queue work

Upload an object the pipeline cannot process, and watch it get retried three
times before landing in the DLQ — the failure path from section 5.4:

```bash
docker compose exec localstack \
  awslocal s3 cp /etc/hostname s3://airport-documents/demo-user-0001/boarding_pass/broken.png

# watch it fail three times
docker compose logs -f worker

# then, after roughly a minute, the message is in the DLQ
docker compose exec localstack \
  awslocal sqs receive-message --queue-url \
  "$(docker compose exec -T localstack awslocal sqs get-queue-url \
     --queue-name airport-document-dlq --output text --query QueueUrl)"
```

{{% notice tip %}}
In AWS, a CloudWatch alarm on the DLQ's `ApproximateNumberOfMessagesVisible`
metric is what pages the operations team at this point.
{{% /notice %}}
