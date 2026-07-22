---
title : "Conclusion"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

#### Summary

The Airport Management System architecture is built on modern AWS foundations, closely following the reference model presented in the [reference architecture section](../5.2-Reference-architecture/) in both technology and deployment approach:

+ **CloudFront + S3** for the presentation tier.
+ **Cognito** for user authentication.
+ **ECS/Fargate behind an Application Load Balancer** for the business API tier.
+ **RDS PostgreSQL Multi-AZ** for data.
+ **SQS + Textract + Lambda** for the OCR processing pipeline.
+ **EventBridge + SES** for the notification system.
+ **CodeBuild + ECR + CodeDeploy** for the automated CI/CD pipeline.

The entire data flow is designed as a closed loop following the caller-to-callee principle, and resources in private subnets connect to S3, Textract, and SES through **VPC Endpoints** — matching AWS network security recommendations and minimizing traffic over the public Internet.

#### Proposed extensions

{{% notice tip %}}
Future improvements worth evaluating:

+ Add cost monitoring with **AWS Budgets / Cost Explorer**.
+ Consider **AWS Backup** for RDS.
+ Assess whether a **NAT Gateway** is needed for tasks outside the scope of VPC Endpoints — for example, calling third-party APIs hosted outside AWS.
{{% /notice %}}
