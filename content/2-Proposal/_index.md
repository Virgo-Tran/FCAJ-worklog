---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Airport Information Management System
## Automating boarding-pass and luggage-tag data entry with a serverless OCR pipeline on AWS

### 1. Executive Summary

Airports handle a large volume of image-based documents every day — boarding
passes, luggage tags, passports. Most of that data is still re-typed by hand
into internal systems, which is slow, error-prone, and impossible to scale
during peak hours.

This proposal describes a three-tier system on AWS that removes the manual step.
A passenger or staff member uploads a document; the system extracts the data
with OCR, reconciles it against the flight schedule, and issues the resulting
notifications automatically.

The design follows the AWS Well-Architected Framework: a public tier holding the
load balancer and VPC endpoints, a private tier holding the application
containers and database, deployed across two Availability Zones. The document
pipeline is event-driven and decoupled by queues, so a burst of uploads is
absorbed rather than dropped, and a document that cannot be read is quarantined
for review instead of failing silently.

A working prototype of the pipeline has been built and verified; section 5 of
this report documents the architecture and section 5.6 walks through running it.

### 2. Problem Statement

#### What's the Problem?

+ **Manual data entry does not scale.** Every boarding pass processed at a desk
  is typed by a person. Peak-hour volume is exactly when staff have the least
  time available.
+ **Typing introduces errors.** A mistyped seat number, gate or passport number
  propagates into downstream systems and is expensive to trace back.
+ **Passengers are notified late, or not at all.** Delay, gate-change and
  cancellation information depends on someone remembering to send it.
+ **There is no single record of a document.** The uploaded image, the extracted
  data, and the boarding pass it belongs to live in different places, so
  disputes are hard to resolve.

#### The Solution

An event-driven pipeline that turns an uploaded image into structured, linked
data with no human keystrokes:

1. The API issues a **presigned URL**; the client uploads straight to Amazon S3,
   so the document bytes never pass through the application tier.
2. The `ObjectCreated` event lands on an **Amazon SQS** queue, decoupling upload
   from processing.
3. A **Lambda** function calls **Amazon Textract** to extract the passenger
   name, flight number, departure and arrival times, seat, gate, boarding time,
   passport number and barcode data.
4. A second **Lambda** writes the result to **Amazon RDS for PostgreSQL**,
   matches it to the correct flight and passenger, and issues the boarding pass.
5. **Amazon SES** sends the passenger a confirmation; **Amazon EventBridge**
   drives a recurring summary of upcoming flights.

Documents that cannot be read are retried, then moved to a **dead letter queue**
that raises a **CloudWatch** alarm, so failures are visible rather than lost.

#### Benefits and Return on Investment

| Benefit | Effect |
|---|---|
| No manual transcription | Staff time redirected from typing to passenger service |
| Fewer data-entry errors | Extraction is deterministic and carries a confidence score |
| Notifications become automatic | Delay, gate change, boarding and cancellation are event-driven |
| Complete document trail | Image, extracted data and boarding pass are linked by foreign key |
| Absorbs peak load | Queues buffer bursts; containers scale on demand |
| Personal data protected | Passport numbers are detected and masked before storage |

The system consumes no fixed capacity when idle: Lambda, SQS, S3 and Textract
bill per use, and the container tier scales with traffic.

### 3. Solution Architecture

![Architecture](/images/5-Workshop/airport-architecture-overview.png)

Traffic enters through Route 53 and CloudFront, is filtered by AWS WAF, and is
routed by an Application Load Balancer to containers running on ECS/Fargate in
private subnets. Those containers reach S3, Textract and SES through VPC
Endpoints, so no traffic leaves the AWS network.

#### AWS Services Used

| Layer | Services |
|---|---|
| **Edge & delivery** | Route 53, CloudFront, AWS WAF |
| **Identity** | Amazon Cognito (JWT; group membership separates passengers from staff) |
| **Compute** | Amazon ECS on AWS Fargate, AWS Lambda |
| **Storage & data** | Amazon S3, Amazon RDS for PostgreSQL (Multi-AZ), Amazon DynamoDB |
| **Integration** | Amazon SQS (+ dead letter queue), Amazon EventBridge |
| **Machine learning** | Amazon Textract |
| **Messaging** | Amazon SES |
| **Networking** | Amazon VPC, Internet Gateway, VPC Endpoints (Gateway and Interface) |
| **Security & config** | AWS KMS, AWS Secrets Manager, AWS Systems Manager Parameter Store, Amazon Macie |
| **Observability** | Amazon CloudWatch and CloudWatch Logs |
| **Delivery pipeline** | AWS CodeBuild, Amazon ECR, AWS CodeDeploy |

#### Component Design

+ **Business API (ECS/Fargate).** Flight, passenger and boarding-pass
  management; issues presigned upload URLs. Stateless, so it scales
  horizontally. Validates the Cognito token on every request and scopes each
  query to the caller.
+ **Submit-OCR function (Lambda).** Consumes the document queue, claims the
  object in DynamoDB so a redelivered message cannot bill Textract twice, calls
  Textract, and publishes the result.
+ **Process-result function (Lambda).** Persists the extraction, masks personal
  data in the raw text, reconciles it against flight and passenger records, and
  queues the notification.
+ **Scheduled function (Lambda + EventBridge).** Sends each passenger a summary
  of their upcoming flights on a monthly schedule.
+ **Relational store (RDS PostgreSQL).** Six normalised tables — airports,
  flights, passengers, documents, boarding passes, notifications — with
  `boarding_passes` as the table joining flight, passenger and document.

### 4. Technical Implementation

**Implementation Phases**

| Phase | Scope |
|---|---|
| 1 — Foundation | VPC with public and private subnets across two AZs, IAM roles, security groups, VPC Endpoints |
| 2 — Data tier | RDS PostgreSQL Multi-AZ, schema and constraints, credentials in Secrets Manager |
| 3 — Application tier | Containerised API, ECS service behind the ALB, Cognito user pool and groups |
| 4 — Document pipeline | S3 bucket and event notification, SQS queues with redrive policy, both Lambda functions, Textract integration |
| 5 — Notifications | SES identity and templates, EventBridge schedule, notification history |
| 6 — Observability | CloudWatch metrics, centralised logs, dead-letter-queue alarm |
| 7 — Delivery | CodeBuild, ECR, CodeDeploy blue/green with automatic rollback |

**Technical Requirements**

+ Two Availability Zones minimum for the load balancer, container service and
  database.
+ All application components in private subnets; only the load balancer is
  publicly reachable.
+ Private-subnet access to S3, Textract and SES through VPC Endpoints rather
  than a NAT Gateway — lower cost and a smaller attack surface.
+ Encryption in transit throughout, and at rest for S3, RDS and Secrets Manager
  using KMS.
+ Every API request authenticated; personal data readable only by its owner or
  by staff.
+ Idempotent document processing, since SQS delivery is at-least-once.

### 5. Timeline & Milestones

Aligned to the thirteen-week internship:

| Weeks | Milestone |
|---|---|
| 1–2 | AWS account governance, IAM, core networking and compute fundamentals |
| 3–4 | DNS, CDN, NoSQL, automation with CloudFormation and CDK, security services |
| 5 | Containers, ECS, workflow orchestration, cost analysis |
| 6–7 | Microservices and event-driven design; serverless application delivery; workshop topic chosen |
| 8 | Production container practice on ECS and Fargate with a CI/CD pipeline |
| 9–10 | Data analytics and machine learning foundations; architecture drafted |
| 11 | Outstanding labs completed; architecture reviewed |
| 12 | Workshop completed; prototype implemented and verified end to end |
| 13 | Report written and submitted |

### 6. Budget Estimation

{{% notice warning %}}
The figures below are my own estimates for a small production deployment in
`ap-southeast-1`, based on published on-demand pricing. They are indicative, not
a quotation — confirm with the AWS Pricing Calculator before committing, and
replace this note with your own saved estimate link.
{{% /notice %}}

#### Infrastructure Costs

Assumptions: 2,000 documents processed per month, 10,000 notification emails,
50 GB of CloudFront egress, two Fargate tasks running continuously.

| Service | Configuration | Est. monthly (USD) |
|---|---|---|
| Amazon Textract | 2,000 pages, forms analysis | ~100.00 |
| Amazon RDS PostgreSQL | db.t4g.micro, Multi-AZ, 20 GB | ~50.00 |
| Amazon ECS on Fargate | 2 tasks, 0.5 vCPU / 1 GB | ~36.00 |
| Application Load Balancer | 1 ALB, low LCU usage | ~20.00 |
| VPC Interface Endpoints | 2 endpoints | ~14.00 |
| AWS WAF | 1 web ACL, managed rule groups | ~10.00 |
| Amazon CloudFront | 50 GB egress | ~5.00 |
| Amazon CloudWatch | Metrics, logs, alarms | ~5.00 |
| Amazon S3 | 50 GB plus requests | ~2.00 |
| SQS, DynamoDB, Lambda, SES, Route 53 | Low volume, largely within free tier | ~5.00 |
| **Total** | | **~247.00** |

**Cost observations**

+ **Textract dominates.** At roughly 40% of the total, document volume is the
  single biggest cost driver. Restricting analysis to the fields actually needed
  is the first lever to pull.
+ **VPC Endpoints are cheaper than a NAT Gateway** at this scale, and remove an
  egress path at the same time.
+ **Single-AZ RDS halves the database cost** and is reasonable for a
  non-production environment, though it forfeits automatic failover.
+ AWS Budgets alerts should be configured before deployment, so spend is capped
  by policy rather than by noticing.

### 7. Risk Assessment

#### Risk Matrix

| Risk | Likelihood | Impact | Rating |
|---|---|---|---|
| OCR misreads a field, producing wrong passenger data | Medium | High | **High** |
| Textract costs exceed budget as volume grows | Medium | Medium | **Medium** |
| Personal data stored unmasked | Low | High | **High** |
| Duplicate processing from at-least-once queue delivery | Medium | Medium | **Medium** |
| Database becomes unavailable | Low | High | **High** |
| Credentials exposed in configuration | Low | High | **High** |
| Traffic spike overwhelms the pipeline | Medium | Low | **Low** |

#### Mitigation Strategies

+ **Extraction accuracy.** Textract returns a per-field confidence score, stored
  alongside the result. Low-confidence extractions can be routed for human
  review rather than committed silently, and the raw text is retained so any
  record can be audited against its source image.
+ **Cost control.** AWS Budgets alerts on the Textract line specifically; a
  CloudWatch metric tracks documents processed, so volume growth is visible
  before the bill arrives.
+ **Personal data.** Uploaded documents are scanned for personal data; passport
  numbers, dates of birth, e-mail addresses and phone numbers are masked in the
  stored text while the structured fields the business needs are retained.
  Amazon Macie classifies the objects in S3 independently.
+ **Duplicate processing.** Every document is claimed in DynamoDB with a
  conditional write before any Textract call, so a redelivered message is
  discarded rather than re-billed.
+ **Availability.** RDS runs Multi-AZ with automatic failover; the container
  service spans two Availability Zones behind the load balancer.
+ **Credentials.** Database credentials come from Secrets Manager, encrypted
  with KMS, and are injected into the task at runtime — never committed to the
  repository or written into an environment file.
+ **Load.** Queues decouple upload from processing, so a spike lengthens the
  queue instead of failing requests.

#### Contingency Plans

+ **Documents that cannot be processed** are retried three times, then moved to
  a dead letter queue that raises a CloudWatch alarm for manual handling. The
  document row records the failure reason.
+ **A bad deployment** is rolled back automatically: CodeDeploy shifts traffic
  only after health checks pass, and reverts the load balancer to the previous
  task set if they do not.
+ **Loss of the database** is covered by automated RDS backups and point-in-time
  recovery; the documents themselves remain in S3 and can be reprocessed.
+ **A Textract outage** leaves documents queued rather than lost; processing
  resumes when the service recovers.

### 8. Expected Outcomes

#### Technical Improvements

+ Manual transcription of boarding passes and luggage tags is eliminated for
  documents the pipeline can read.
+ Flight notifications become a property of the system rather than a task
  someone must remember.
+ Every uploaded document has a durable, linked record: the image in S3, the
  extracted data in PostgreSQL, and the boarding pass that references both.
+ Failures are visible. A document that cannot be read raises an alarm instead
  of disappearing.

#### Long-term Value

+ **A foundation for further automation.** The same pipeline shape — presigned
  upload, queue, extract, reconcile, notify — extends to passports, visas and
  identity documents with no architectural change.
+ **A dataset worth analysing.** Processed documents accumulate into a corpus
  suitable for analytics on extraction accuracy, peak-hour volume and
  notification effectiveness.
+ **Operational maturity.** Idempotency, dead-letter handling, blue/green
  deployment and per-caller authorisation are transferable patterns, not one-off
  features of this system.
+ **Cost transparency.** Because the workload is event-driven, cost scales with
  documents processed, making the economics of automation directly measurable.
