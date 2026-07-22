---
title : "Prerequisites and start-up"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1 </b> "
---

#### Prerequisites

+ **Docker Desktop** (or any Docker with Compose v2). Nothing else is required —
  Python, PostgreSQL and the AWS CLI all run inside containers.
+ Roughly 2 GB of free disk for the images.
+ Ports **8000** (API), **5432** (PostgreSQL) and **4566** (LocalStack) free.

{{% notice note %}}
No AWS account is needed and nothing is billed. Amazon S3 and Amazon SQS are
emulated by LocalStack; Textract and SES are replaced by local providers.
{{% /notice %}}

#### Start the stack

```bash
cd TranQuocKhanh-Project
docker compose up --build
```

The first run takes a few minutes. Compose brings up four services in order:

1. **postgres** — creates the schema and seed data from `db/*.sql` on first boot.
2. **localstack** — creates the S3 bucket, the three queues, the DLQ redrive
   policy, and the S3 → SQS event notification.
3. **api** — waits for both to report healthy, then serves on port 8000.
4. **worker** — starts polling the document and OCR-results queues.

Wait for the API to report healthy:

```bash
curl http://localhost:8000/health
# {"status":"ok","database":"ok"}
```

#### Explore the API

Open <http://localhost:8000/docs> for the generated Swagger UI. The seed data
gives you four airports, three flights and three passengers to work with:

```bash
curl http://localhost:8000/flights
curl http://localhost:8000/airports
```

#### Useful commands

| What | Command |
|---|---|
| Worker logs | `docker compose logs -f worker` |
| Database shell | `docker compose exec postgres psql -U airport -d airport` |
| List queues | `docker compose exec localstack awslocal sqs list-queues` |
| List uploaded objects | `docker compose exec localstack awslocal s3 ls s3://airport-documents --recursive` |
