---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Airport Information Management System on AWS

#### Overview

This workshop walks through the architecture of an **Airport Information Management System** — a system serving two groups of users, passengers and airport staff, that handles authentication, flight and passenger management, automatic data extraction from boarding passes and luggage tags via OCR, and automated flight notifications.

The system is built as a modern **three-tier web architecture** on AWS that combines two execution models:

+ **Serverless** (Lambda, S3, CloudFront) for event-driven tasks.
+ **Containers** (ECS on AWS Fargate) for the core business API tier.

On top of these sit an **OCR document processing pipeline** built on Amazon Textract and a **CI/CD pipeline** that automates deployment.

The architecture follows the **AWS Well-Architected Framework**: public subnets (ALB, VPC Endpoints) are separated from private subnets (application containers, database), and everything is deployed across at least **two Availability Zones** (AZ-1, AZ-2) for high availability.

#### Content

1. [Introduction](5.1-Introduction/)
2. [Reference architecture (AWS V2)](5.2-Reference-architecture/)
3. [Airport system architecture](5.3-Project-architecture/)
4. [OCR document pipeline](5.4-OCR-pipeline/)
5. [Database design](5.5-Database-design/)
6. [Running the prototype](5.6-Prototype/)
7. [Conclusion](5.7-Conclusion/)
