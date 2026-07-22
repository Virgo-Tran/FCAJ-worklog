---
title : "AWS services used"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

The project uses **18 AWS services**, applying the same foundational technology as the V2 reference architecture with roles customized for airport management:

| # | AWS service | Category | Role in the architecture |
|---|---|---|---|
| 1 | **Route 53** | Networking & Content Delivery | Resolves `airport.example.com`, bringing users (passengers and staff) to the nearest CloudFront edge. |
| 2 | **CloudFront** | Networking & Content Delivery | Distributes the Airport Web UI and routes dynamic API requests down to the ALB. |
| 3 | **WAF** | Security, Identity & Compliance | Protects the system against web attacks while absorbing high passenger traffic volumes. |
| 4 | **Amazon S3** | Storage | Hosts the static web frontend and stores documents uploaded by passengers and staff: boarding passes and luggage tags. |
| 5 | **Amazon Cognito** | Security, Identity & Compliance | Authenticates airport staff and passengers, issues JWTs, and supplies the Owner ID attached to each uploaded document. |
| 6 | **AWS Lambda** | Compute (Serverless) | Three functions for three distinct jobs: **Get OwnerID Service** (owner lookup), **Submit Job** (trigger OCR), and **Process OCR Result / Process Scheduled Tasks** (write OCR results, send periodic notifications). |
| 7 | **Amazon VPC, IGW & VPC Endpoint** | Networking & Content Delivery | Network isolation for all airport business services; VPC Endpoints let ECS and Lambda reach S3, Textract, and SES securely without going out to the public Internet. |
| 8 | **Application Load Balancer** | Networking & Content Delivery | Distributes flight and passenger management API requests across Fargate tasks in two Availability Zones. |
| 9 | **Amazon ECS on AWS Fargate** | Compute (Container) | Runs the airport business APIs (flight, passenger, boarding pass management) as containers, auto-scaling with peak-hour traffic. |
| 10 | **Amazon RDS (PostgreSQL)** | Database | Stores relational data: airports, flights, passengers, boarding passes, documents, notifications — with a Standby for failover. |
| 11 | **Amazon SQS** | Application Integration | **SQS Document Processing** (uploaded-document queue), **Dead-Letter Queue** (failed messages), and **SQS OCR Results** (OCR output awaiting processing). |
| 12 | **Amazon Textract** | Machine Learning | Automatically extracts passenger name, flight number, departure/arrival time, seat number, gate, passport number, and more from boarding pass images. |
| 13 | **Amazon EventBridge** | Application Integration | Fires monthly so the system can aggregate and send flight notifications to passengers. |
| 14 | **Amazon SES** | Business Applications | Sends flight notification emails: delays, gate changes, boarding reminders, cancellations, and monthly summaries. |
| 15 | **Amazon CloudWatch (+ Logs)** | Management & Governance | Monitors ECS activity, centralizes logs, and alarms when OCR document processing fails into the DLQ. |
| 16 | **AWS CodeBuild** | Developer Tools | Builds Docker images for the airport management services from source on GitHub. |
| 17 | **Amazon ECR** | Developer Tools | Stores versioned container images of the airport services. |
| 18 | **AWS CodeDeploy** | Developer Tools | Automatically deploys approved new service versions onto the ECS Cluster. |

{{% notice note %}}
The reference architecture lists 19 services while the project uses 18 — the project consolidates the Lambda-related entries into a single row covering all three functions.
{{% /notice %}}
