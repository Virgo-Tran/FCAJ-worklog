---
title : "Airport system architecture"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Applying the reference architecture

The Airport Information Management System is built on the AWS V2 reference architecture from the [previous section](../5.2-Reference-architecture/). It applies the same foundational technology — ECS/Fargate, RDS PostgreSQL, SQS, Textract, EventBridge, SES — with each service's role tailored to the aviation domain.

#### Architecture diagram

![Airport Management System architecture](/images/5-Workshop/airport-architecture-overview.png)

*Figure 2.1 — Overall architecture of the Airport Management System.*

#### Content

- [AWS services used](5.3.1-services/)
- [Data flow](5.3.2-dataflow/)
