---
title : "OCR document pipeline"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

This is the most important domain-specific workflow in the system: automatically extracting data from boarding passes and luggage tags uploaded by staff or passengers, eliminating manual data entry.

![OCR document processing pipeline](/images/5-Workshop/ocr-pipeline.png)

*Figure 2.2 — OCR document pipeline: Textract → SQS OCR Results → Lambda Process OCR Result → RDS → SES.*

#### Pipeline steps

1. Airport staff request a presigned URL; **Lambda Gen Presigned URL** generates it.
2. The document is uploaded directly to the **S3 Airport Docs** bucket.
3. S3 emits an `Object Created` event into the **SQS Document Queue**.
4. **Lambda Submit OCR Job** polls the queue and calls **Amazon Textract** via a VPC Endpoint.
5. Textract completes OCR and the result is pushed to **SQS OCR Results**.
6. **Lambda Process OCR Result** polls that queue, saves the extracted data to **RDS PostgreSQL**, and sends a notification through **SES**.

#### Fields extracted from a boarding pass

Textract extracts the following key fields:

+ Passenger name
+ Flight number
+ Departure / arrival time
+ Seat number
+ Gate
+ Boarding time
+ Passport number
+ Barcode data

#### Failure handling

{{% notice warning %}}
Messages that fail processing after exceeding the retry limit are moved to the **Dead Letter Queue**, and **CloudWatch** raises an alarm so the operations team can intervene.
{{% /notice %}}
