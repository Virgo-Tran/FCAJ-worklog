---
title : "Data flow"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

The main processing flow consists of the following steps:

**Step 1 — DNS resolution.** Route 53 resolves `airport.example.com` and redirects the user to CloudFront.

**Step 2 — Serve the UI.** CloudFront serves the airport management web UI (Airport Web UI) from the S3 Static Frontend.

**Step 3 — Authentication.** The user (passenger or staff member) authenticates through Cognito and receives a JWT.

**Step 4 — API routing.** CloudFront and the ALB route API requests (flight management, passenger management, and so on) to the ECS Cluster.

**Step 5 — Authorization.** The ECS Cluster validates the JWT directly against Cognito and calls the **Lambda Get OwnerID Service** — which queries the Cognito Admin API — to attach an Owner ID to the request.

**Step 6 — Business data.** The ECS Cluster reads and writes business data (flights, passengers) to RDS PostgreSQL.

**Step 7 — Document upload.** The ECS Cluster generates a presigned URL (through the S3 Gateway Endpoint); staff or passengers upload a boarding pass or luggage tag directly to the S3 Airport Documents bucket.

**Step 8 — OCR trigger.** S3 emits an `Object Created` event → **SQS Document Processing** → **Lambda Submit Job** → calls Amazon Textract to run OCR, extracting name, flight number, seat number, gate, passport number, and so on.

**Step 9 — OCR result handling.** Textract completes and pushes results onto **SQS OCR Results**; **Lambda Process OCR Result** reads the queue, writes the extracted data into RDS, and sends a notification through SES (via a VPC Endpoint).

**Step 10 — Scheduled notifications.** EventBridge fires monthly → **Lambda Process Scheduled Tasks** queries RDS for flight data → sends bulk notifications through SES.

**Step 11 — Failure handling.** Messages that exceed the retry limit land in the **Dead-Letter Queue** → CloudWatch raises an alarm so the operations team can investigate and resolve them manually.
