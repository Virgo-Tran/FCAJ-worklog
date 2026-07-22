---
title : "Clean up"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.6.3 </b> "
---

#### Stop the stack

```bash
cd TranQuocKhanh-Project
docker compose down
```

This stops and removes the containers but keeps the database volume, so the
next `docker compose up` resumes with your data intact.

#### Remove everything

To also drop the PostgreSQL volume — the next start will recreate the schema
and re-seed:

```bash
docker compose down -v
```

To reclaim the disk used by the built images as well:

```bash
docker compose down -v --rmi local
```

{{% notice note %}}
Nothing in this prototype runs on AWS, so there is no cloud spend to stop and
no AWS resources to delete. If you switched `OCR_PROVIDER=textract` or
`NOTIFIER=ses`, those calls did go to a real account — check the Textract and
SES usage in Cost Explorer.
{{% /notice %}}

#### What to build next

The prototype deliberately stops short of the full architecture. The gaps, in
rough order of value:

1. **Cognito authentication.** The API currently takes `owner_id` as a
   parameter; the deployed system validates a JWT and reads the `sub` claim.
   There is no authorisation at all, so never expose this beyond localhost.
2. **The EventBridge monthly job.** Step 10 of the data flow — a scheduled
   Lambda that queries RDS and sends bulk notifications — is not implemented.
3. **Infrastructure as Code.** A CDK or Terraform stack to provision the real
   18 services.
4. **A frontend.** The S3 + CloudFront static UI the passengers and staff
   actually use.
5. **Better reconciliation.** Matching is exact on flight number plus a
   case-insensitive name; real data needs fuzzier matching and a human review
   path for low-confidence extractions.
