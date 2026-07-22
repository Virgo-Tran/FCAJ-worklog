---
title : "Introduction"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### The problem

Airports handle a large volume of paper and image-based documents every day: boarding passes, luggage tags, passports. Today most of that data is re-typed by hand into internal systems — slow, error-prone, and impossible to scale during peak hours.

The **Airport Information Management System** addresses this by serving two groups of users:

+ **Passengers** — check flight information, upload their boarding pass, and receive notifications.
+ **Airport staff** — manage flights and passenger records, and process uploaded documents.

#### Capabilities

The system provides:

1. **User authentication** — passengers and staff sign in and receive a JWT.
2. **Flight and passenger management** — full CRUD over airports, flights, and passenger records.
3. **Automatic OCR extraction** — upload a boarding pass or luggage tag and the system extracts passenger name, flight number, departure/arrival time, seat number, gate, passport number, and barcode data with no manual typing.
4. **Automated notifications** — email alerts for delays, gate changes, boarding reminders, cancellations, and a monthly summary.

#### Approach

The project is built on the **AWS V2 reference architecture** described in the [next section](../5.2-Reference-architecture/). It reuses the same foundational technology — ECS/Fargate, RDS PostgreSQL, SQS, Textract, EventBridge, SES — but customizes each service's role for the specific needs of the aviation domain.

{{% notice note %}}
This workshop is organized in two parts, mirroring how the system was designed: first the **generic reference architecture** that establishes the patterns, then the **concrete airport system** that applies them.
{{% /notice %}}
