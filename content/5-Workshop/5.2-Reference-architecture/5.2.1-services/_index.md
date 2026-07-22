---
title : "AWS services used"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.2.1 </b> "
---

The reference architecture uses a total of **19 AWS services** spanning several categories: Networking & Content Delivery, Security, Compute, Database, Application Integration, Machine Learning, Management & Governance, and Developer Tools (CI/CD).

| # | AWS service | Category | Role in the architecture |
|---|---|---|---|
| 1 | **Route 53** | Networking & Content Delivery | DNS service; resolves the domain name (e.g. `airport.example.com`) and routes users to CloudFront at the edge. |
| 2 | **CloudFront** | Networking & Content Delivery | Global CDN and the single entry point for all traffic; caches and serves the static web UI while forwarding dynamic API requests down to the ALB. |
| 3 | **WAF** | Security, Identity & Compliance | Attached to CloudFront to filter and block common web attacks (SQL injection, XSS, scanning bots) before requests reach the backend. |
| 4 | **Amazon S3** | Storage | Stores the static frontend assets and the files users upload (Image S3) via presigned URLs. |
| 5 | **Amazon Cognito** | Security, Identity & Compliance | Manages user identity, authenticates users, and issues the JSON Web Tokens (JWT) used for subsequent API calls. |
| 6 | **AWS Lambda** | Compute (Serverless) | Runs event-driven functions with no servers to manage: Owner ID lookup, OCR job submission, and scheduled tasks. |
| 7 | **Amazon VPC & Internet Gateway** | Networking & Content Delivery | Virtual private network isolating all backend resources; the Internet Gateway lets public-subnet resources (ALB) communicate with the Internet. |
| 8 | **VPC Endpoint** (Gateway & Interface) | Networking & Content Delivery | Lets private-subnet resources (ECS, Lambda) reach S3, Textract, and SES over the AWS internal network — no NAT Gateway and no trip through the public Internet. |
| 9 | **Elastic Load Balancing (ALB)** | Networking & Content Delivery | Application Load Balancer distributing API requests across containers running in multiple Availability Zones. |
| 10 | **Amazon ECS on AWS Fargate** | Compute (Container) | Runs the backend service as serverless containers (no EC2 to manage), auto-scaling with load, placed in private subnets. |
| 11 | **Amazon RDS (PostgreSQL)** | Database | The primary relational database, deployed Primary/Standby (Multi-AZ) for high availability and automatic failover. |
| 12 | **Amazon SQS** | Application Integration | Message queues that decouple document upload from OCR processing; includes a dedicated Dead-Letter Queue for failed messages and a separate queue for OCR results. |
| 13 | **Amazon Textract** | Machine Learning | AI service that extracts text and structured data (OCR) from uploaded document images. |
| 14 | **Amazon EventBridge** | Application Integration | Cron-like scheduling that triggers recurring tasks automatically, e.g. sending monthly notifications. |
| 15 | **Amazon SES** | Business Applications | Sends notification emails to end users (processing results, alerts, periodic notices). |
| 16 | **Amazon CloudWatch (+ Logs)** | Management & Governance | Centralized log collection, metrics monitoring, and alarms when messages land in the Dead-Letter Queue or the system errors. |
| 17 | **AWS CodeBuild** | Developer Tools | Compiles the source code and builds the Docker image for the backend service. |
| 18 | **Amazon ECR** | Developer Tools | Stores and versions the built Docker images, serving as the image source for CodeDeploy. |
| 19 | **AWS CodeDeploy** | Developer Tools | Automatically deploys the newest container version from ECR onto the ECS Cluster, supporting rolling deployments. |
