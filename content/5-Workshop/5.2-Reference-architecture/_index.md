---
title : "Reference architecture (AWS V2)"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### What AWS V2 is

**AWS V2** is the reference architecture used as the design foundation for the Airport Information Management System. It is a modern three-tier web architecture on AWS that combines a **serverless** model (Lambda, S3, CloudFront) for event-driven tasks with a **containerized** model (ECS/Fargate) for the core business API tier, plus an OCR document pipeline (Amazon Textract) and an automated CI/CD deployment pipeline.

The architecture is organized according to the **AWS Well-Architected Framework**: public subnets (holding the ALB and VPC Endpoints) are separated from private subnets (holding the application containers and the database), deployed across a minimum of two Availability Zones (AZ-1, AZ-2) to guarantee high availability.

#### Architecture diagram

![AWS V2 reference architecture](/images/5-Workshop/aws-v2-reference-architecture.png)

*Figure 1.1 — AWS V2 reference architecture, including Route 53 and the DEV/CI-CD pipeline.*

#### Content

- [AWS services used](5.2.1-services/)
- [Data flow](5.2.2-dataflow/)
- [CI/CD pipeline](5.2.3-cicd/)
