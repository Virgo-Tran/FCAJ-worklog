---
title: "Blog 2: AWS Compute Decision Framework"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Lambda, ECS Fargate, EKS, or EC2 — Which One Should You Actually Use?

> **Category:** *AWS Compute Decision Framework*  
> **Author:** Tran Quoc Khanh  
> **Community:** [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj)  

*A practical decision framework for architects and backend engineers who are tired of guessing and ready to pick the right compute platform for every workload.*

---

If you've ever sat through an AWS architecture review, chances are one question kept coming back: *"Should this run on Lambda, ECS, EKS, or EC2?"* This isn't an academic question — it directly affects your operating costs, how fast your team ships, and how much operational overhead your team takes on.

AWS offers at least six core compute options, each optimized for a different kind of workload. This post walks through a practical framework based on traffic patterns, latency requirements, team capacity, and cost.

---

### 1. The Core Trade-off: Control vs. Operational Overhead

Every AWS compute option sits on the same trade-off curve: **The more infrastructure control you have, the more operational overhead you take on.**

* **Amazon EC2:** Full control over virtual machines, OS, runtime, patching, and auto-scaling. In exchange, your team owns every operational task.
* **AWS Fargate (on Amazon ECS):** Removes server management burdens. You work at the container level while AWS manages the underlying OS and cluster.
* **Amazon EKS:** Fully-featured managed Kubernetes. Excellent for large microservice architectures, but demands high operational maturity.
* **AWS Lambda:** Fully serverless model. Focus purely on writing function code; AWS handles provisioning and scaling automatically.
* **AWS App Runner:** Radically simplified container service for always-on web apps with minimal infrastructure configuration.
* **AWS Batch:** Purpose-built for discrete batch compute jobs. Automatically provisions resources based on job queue state.

---

### 2. Deep Dive into Primary Compute Services

#### AWS Lambda — Built for Event-Driven Architectures
Ideal for processing S3 uploads, SQS messages, API Gateway requests, or DynamoDB streams. Its pay-per-use model provides ultimate cost efficiency for bursty, unpredictable traffic.

* **Key limits:** 15-minute execution cap; cold start latency; 1,000 default concurrency limit; no GPU support.

#### Amazon ECS Fargate — The Sweet Spot Between Lambda and EKS
Fargate is right when workloads run 24/7, maintain in-memory state, require custom system libraries, or have high predictable traffic (over 50-100 million requests/month, Fargate flat pricing beats Lambda per-invocation costs).

#### Amazon EKS — Serious Power for Scale
EKS should be a priority in three cases: engineering teams already experienced with production Kubernetes; multi-cloud portability requirements; or deep integration needs with Kubernetes-native tooling (Helm, Istio, GitOps).

#### Amazon EC2 — Full Control for Specialized Workloads
Optimal for GPU-intensive machine learning workloads (p4d, g5, p5), persistent connections (WebSockets, game servers), legacy applications, or steady massive traffic using Reserved Instances / Savings Plans.

---

### 3. Quick-Reference Guide by Real-World Scenario

| Application Scenario | Recommended Compute Service | Technical Reasoning |
| :--- | :--- | :--- |
| **Event webhook receiver** (S3, GitHub, Stripe) | **AWS Lambda** | Short-lived, event-driven task; near-zero cost when idle. |
| **Low or unpredictable REST API traffic** | **AWS Lambda** or **AWS App Runner** | Scale to zero capability; App Runner suits simple always-on apps. |
| **High, steady REST API traffic** | **Amazon ECS Fargate** or **Amazon EC2** | Lambda per-request pricing stops being cost-effective at scale. |
| **API requiring sub-100ms p99 latency** | **Amazon ECS Fargate** / **AWS App Runner** | Avoids latency volatility caused by Lambda cold starts. |
| **Scheduled job running under 15 mins** | **AWS Lambda + Amazon EventBridge** | No standing infrastructure cost between runs. |
| **Scheduled job running over 15 mins** | **AWS Batch** or **Amazon ECS Task** | Bypasses Lambda 15-minute execution limit. |
| **Stateful WebSocket / real-time connections** | **Amazon ECS Fargate** or **Amazon EC2** | Lambda isn't designed to hold persistent connections. |
| **ML training requiring GPU resources** | **Amazon EC2 (GPU)** or **AWS Batch** | Lambda and Fargate don't support GPU acceleration. |
| **Microservices system (5-20 services)** | **Amazon ECS Fargate** | Production-grade without K8s cluster overhead. |
| **Large-scale, multi-cloud microservices** | **Amazon EKS** | Preserves consistency of Kubernetes-native tooling. |

---

### 4. Total Cost of Ownership (TCO): Looking Beyond Compute Bills

When calculating TCO for a reference workload, look beyond raw compute charges:
* **Staffing costs:** Especially engineering time spent managing EKS clusters.
* **Supporting infrastructure:** Fixed ALB fees, ECR container image storage, CloudWatch log storage.
* **Data Transfer Out fees:** A quiet but significant cost across AWS services.

---

### 5. The 6-Question Decision Framework

1. *Is the workload event-driven and completes in under 15 mins?* **Yes → AWS Lambda.**
2. *Is traffic stable or volatile?* **Volatile → AWS Lambda.** **Stable → Lambda Managed Instances, ECS Fargate, or EKS.**
3. *Does the workload need GPU, deep OS kernel access, or persistent storage?* **Yes → Amazon EC2.**
4. *Does the system require Kubernetes tooling or multi-cloud portability?* **Yes → Amazon EKS.** **No → Amazon ECS Fargate.**
5. *Does the job run longer than 15 mins with a scheduling requirement?* **Yes → AWS Batch.**
6. *Does the team have deep K8s expertise and under 10 services?* **No K8s expertise → Amazon ECS Fargate.**

---

### 6. Conclusion: The Best Production Architecture Is Usually Hybrid

Few large production systems rely on a single compute service. A typical hybrid model combines **API Gateway + Lambda** for webhooks, **ECS Fargate** for core services, and **AWS Batch** for heavy end-of-day batch processing.

