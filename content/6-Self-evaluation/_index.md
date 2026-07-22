---
title: "Self-Assessment"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

During my internship at **Amazon Web Services Viet Nam Company Limited** from
**05/05/2026** to **30/07/2026** — thirteen weeks on the Workforce Bootcamp,
First Cloud AI Journey — I had the opportunity to learn, practice, and apply the
knowledge acquired at school in a real working environment.

The programme moved from account fundamentals through to a delivered project. In
the first weeks I established account governance with AWS Budgets and
least-privilege IAM, then built the core network and compute layer: a VPC with
public and private subnets, EC2, S3 and RDS. From there I worked through DNS and
content delivery with Route 53 and CloudFront, NoSQL modelling in DynamoDB, and
the AWS migration and disaster-recovery toolchain.

The middle of the programme covered automation and operations — Lambda,
CloudFormation and the AWS CDK for repeatable infrastructure, Systems Manager
and Grafana for day-to-day management — alongside the core security services:
WAF, GuardDuty, Security Hub, Macie and encryption with KMS. I then moved into
containers and workflow orchestration with Docker, Amazon ECS and Step
Functions, followed by microservice decomposition, event-driven design, and a
serverless CRUD application traced with AWS X-Ray. The later weeks added
production container skills on ECS and EKS, data analytics with Glue, Athena and
QuickSight, and the machine learning lifecycle on Amazon SageMaker.

My main deliverable was the **Airport Information Management System**, a
three-tier architecture on AWS built to automate a task airports still perform
by hand: reading boarding passes and luggage tags. I designed the architecture,
documented it as a bilingual workshop, and implemented a working prototype of
the document pipeline — presigned upload to S3, an event-driven queue, OCR
extraction, reconciliation against the flight schedule in PostgreSQL, and
automated passenger notifications, with a dead letter queue and CloudWatch alarm
covering the failure path.

Through this work I improved my skills in **cloud architecture design, network
and identity security, containerisation, event-driven and serverless
development, relational data modelling, infrastructure as code, technical
writing in two languages, and systematic testing and verification**.

To reflect objectively on my internship, I evaluate myself against the following
criteria:

| No. | Criteria | Description | Good | Fair | Average |
| --- | -------- | ----------- | ---- | ---- | ------- |
| 1   | **Professional knowledge & skills** | Understanding of the field, applying knowledge in practice, proficiency with tools, work quality | ☑ | ☐ | ☐ |
| 2   | **Ability to learn** | Ability to absorb new knowledge and learn quickly | ☑ | ☐ | ☐ |
| 3   | **Proactiveness** | Taking initiative, seeking out tasks without waiting for instructions | ☑ | ☐ | ☐ |
| 4   | **Sense of responsibility** | Completing tasks on time and ensuring quality | ☑ | ☐ | ☐ |
| 5   | **Discipline** | Adhering to schedules, rules, and work processes | ☑ | ☐ | ☐ |
| 6   | **Progressive mindset** | Willingness to receive feedback and improve oneself | ☑ | ☐ | ☐ |
| 7   | **Communication** | Presenting ideas and reporting work clearly | ☑ | ☐ | ☐ |
| 8   | **Teamwork** | Working effectively with colleagues and participating in teams | ☑ | ☐ | ☐ |
| 9   | **Professional conduct** | Respecting colleagues, partners, and the work environment | ☑ | ☐ | ☐ |
| 10  | **Problem-solving skills** | Identifying problems, proposing solutions, and showing creativity | ☑ | ☐ | ☐ |
| 11  | **Contribution to project/team** | Work effectiveness, innovative ideas, recognition from the team | ☑ | ☐ | ☐ |
| 12  | **Overall** | General evaluation of the entire internship period | ☑ | ☐ | ☐ |

{{% notice note %}}
The self-assessment ratings above have been completed with **Good** ratings for all criteria, reflecting dedicated effort and achievements throughout the internship.
{{% /notice %}}

### Needs Improvement

The following are drawn from the actual state of my project work, and are the
gaps I intend to close next:

* **Deploying to a live AWS account.** My prototype runs entirely against
  emulated services. I have not yet operated the architecture on real
  infrastructure, where cost, IAM policy detail and regional behaviour all
  matter in a way they do not locally.
* **Running a real delivery pipeline.** I wrote the CodeBuild, ECR and
  CodeDeploy definitions, but never executed them. Watching a blue/green
  deployment roll back on a failed health check is something I have read about
  rather than seen.
* **Testing earlier, and less by inspection.** Two significant defects in my
  project — a firewall rule that stopped the edge proxy from starting, and an
  endpoint that returned passenger data without a token — survived every static
  review and only appeared the first time I ran the whole system. I should build
  and run sooner instead of trusting a careful read.
