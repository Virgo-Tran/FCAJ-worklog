---
title : "Data flow"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2.2 </b> "
---

The main processing flow of the system breaks down into the following steps:

**Step 1** — The user requests the domain name; Route 53 resolves DNS and redirects to CloudFront.

**Step 2** — CloudFront serves the static web UI from S3 (Static Frontend). WAF inspects and filters malicious traffic before it reaches CloudFront.

**Step 3** — The user authenticates through Cognito and receives a JWT. Cognito integrates with the **Lambda Get OwnerID Service** to look up the Owner ID.

**Step 4** — CloudFront forwards dynamic API requests down to the Application Load Balancer (ALB) in the public subnet.

**Step 5** — The ALB distributes requests to ECS containers running on AWS Fargate, placed in private subnets spread across two Availability Zones.

**Step 6** — ECS/Fargate reads and writes business data to RDS PostgreSQL (Primary); data replicates to the Standby instance in real time.

**Step 7** — The user uploads a file to S3 (Image S3) through a presigned URL issued by the backend; the `Object Created` event is pushed onto SQS.

**Step 8** — **Lambda Submit Job** consumes the message from SQS and calls Amazon Textract (via a VPC Endpoint) to run OCR on the document.

**Step 9** — When Textract finishes, it pushes results onto the **SQS OCR Results** queue; **Lambda Process** reads that queue and writes the results into RDS.

**Step 10** — EventBridge triggers Lambda Process on a schedule to send notifications through SES (via a VPC Endpoint).

**Step 11** — The full CI/CD flow (GitHub → CodeBuild → ECR → CodeDeploy) automatically builds, packages, and deploys new versions onto the ECS Cluster; CloudWatch monitors the whole system and centralizes logging.

{{% notice tip %}}
The entire flow is designed as a closed loop following the caller-to-callee principle, and every resource in a private subnet reaches S3, Textract, and SES through VPC Endpoints — minimizing exposure to the public Internet.
{{% /notice %}}
