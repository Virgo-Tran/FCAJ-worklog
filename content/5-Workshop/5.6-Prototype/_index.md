---
title : "Running the prototype"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### From architecture to running code

The previous sections describe the design. This section runs it.

The source code ships alongside this report as a separate package,
**`TranQuocKhanh-Project`**. It is a working implementation of the OCR pipeline
and the business API, built so it can run entirely on your laptop — **no AWS
account and no spend**. Amazon S3 and Amazon SQS are provided by
[LocalStack](https://localstack.cloud/), PostgreSQL runs in a container in place
of RDS, and the two Lambda functions run as ordinary Python processes.

Extract `TranQuocKhanh-Project.zip` to follow along; every path below is
relative to the extracted `TranQuocKhanh-Project/` folder.

#### How the code maps onto the architecture

| Prototype | Deployed architecture |
|---|---|
| `edge/` nginx | Route 53 → CloudFront → WAF → ALB |
| `api/` container | ECS/Fargate task behind an ALB |
| `worker/src/handlers.py` | Two Lambda functions with SQS event source mappings |
| `worker/src/scheduled.py` | A Lambda on an EventBridge cron schedule |
| `worker/src/poller.py` | The Lambda service's own SQS polling — no AWS equivalent |
| `shared/auth.py` dev mode | Amazon Cognito user pool (RS256 via JWKS) |
| `shared/idempotency.py` | Amazon DynamoDB with a conditional write |
| `shared/observability.py` | Amazon CloudWatch metrics and alarms |
| `shared/pii.py` local scan | Amazon Macie classification jobs |
| `shared/parameters.py` | SSM Parameter Store + Secrets Manager (KMS) |
| LocalStack | S3, SQS, DynamoDB, CloudWatch, EventBridge, SSM, Secrets Manager, KMS |
| `ocr.py` mock provider | Amazon Textract |
| `notifier.py` log provider | Amazon SES |
| `postgres` container | Amazon RDS for PostgreSQL, Multi-AZ |
| compose `private` network | Private subnets with no route to the internet |
| `shared/aws.py` `endpoint_url` | VPC Endpoints |
| `buildspec.yml` / `appspec.yml` | CodeBuild → ECR → CodeDeploy (blue/green) |

{{% notice note %}}
The pipeline stages are written as Lambda handlers — they take an SQS-shaped
`event` and return a partial-batch response — so the same code can be lifted
onto real Lambda unchanged. Only the `endpoint_url` in `shared/aws.py` differs
between local and deployed.
{{% /notice %}}

#### Authentication

Locally `AUTH_MODE=dev` verifies HS256 tokens signed with a shared secret,
because LocalStack's community edition has no Cognito; in AWS
`AUTH_MODE=cognito` verifies RS256 tokens against the user pool's JWKS
endpoint. Both perform real signature, issuer and expiry checks, and the
application only ever sees the `sub` claim as `owner_id`.

| Endpoint | Access |
|---|---|
| `/health`, `/docs` | Public |
| `GET /flights`, `GET /airports` | Public — the same schedule a departure board shows |
| `GET /documents`, `GET /notifications` | Token required; scoped to the caller |
| `GET /boarding-passes`, `GET /passengers` | Token required; staff see all, passengers only their own |
| `PATCH /flights/{id}/status`, `POST /passengers` | Token **and** `staff` group |

Flight schedules are deliberately public — a passenger checking a departure
time should not need an account. Everything carrying personal data (names, seat
numbers, passport numbers, barcodes) requires authentication.

{{% notice warning %}}
`SERVICES.md` in the project package is the authoritative coverage list. It
records which services are **verified by tests**, which are **config only** —
written and syntax-checked but never executed — and which services from the
worklog were **deliberately excluded**, with the reasoning for each.
{{% /notice %}}

#### Content

- [Prerequisites and start-up](5.6.1-run/)
- [Walking the OCR pipeline](5.6.2-pipeline/)
- [Clean up](5.6.3-cleanup/)
