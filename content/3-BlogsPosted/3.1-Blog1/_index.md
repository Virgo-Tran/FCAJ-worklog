---
title: "Blog 1: Optimizing Amazon EKS Costs"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Optimizing Amazon EKS Costs: From Bill Shock to 40-60% Savings

> **Category:** *Container Infrastructure Cost Optimization*  
> **Author:** Tran Quoc Khanh  
> **Community:** [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj)  

*Why almost every EKS cluster is overpaying, and a 7-step roadmap to take back control of your budget.*

---

If you've ever opened your AWS bill at the start of the month and found your EKS line item way higher than expected, you're not alone. It's probably the single most common complaint in DevOps communities: teams adopt EKS for Kubernetes' standardization and power, only to discover months later that most of that spend is feeding nodes that are barely doing anything.

A 2026 Kubernetes optimization report surfaced a sobering number: **the average cluster only uses about 8% of allocated CPU and 20% of allocated memory**, while CPU over-provisioning has climbed to 69% year over year. In other words, a large chunk of your EKS budget is likely paying for idle capacity, not real traffic.

---

### 1. The Real Cost Structure of EKS — It's Not Just $0.10/Hour

Many newcomers to EKS only remember the familiar number: **$0.10/hour** per control plane, roughly $73/month. But that's just the tip of the iceberg. The real cost of an EKS cluster stacks up across several layers:

* **Worker Node Compute:** The biggest cost driver, coming from the EC2 instances running your nodes, or vCPU/memory charges if you're running pods on Fargate.
* **Control Plane Fee:** $0.10/hour/cluster, plus an extended support fee of **$0.60/hour** if your cluster is running a Kubernetes version past its standard support lifecycle — that alone can add **$432/month** per cluster running 24/7.
* **EKS Auto Mode Fee:** If you enable fully automated compute management, you pay an additional per-second Auto Mode fee for every EC2 instance it provisions.
* **Networking:** Load Balancers, NAT Gateways, and especially data transfer fees between Availability Zones or out to the internet.
* **Storage:** EBS volumes for persistent storage, plus log storage in CloudWatch.

---

### 2. Case Study: A Small Team's $10,000 Mistake

A widely-shared story in the engineering community illustrates this perfectly: A team migrated to EKS Auto Mode, drawn in by the promise of 'no more infrastructure worries.' A few weeks later, they discovered a single service was costing around **$240/month** — while actual usage needed only a fraction of that. Scaled across their whole system, the estimated waste came out to over **$1,100/month**, purely from resource requests being set far higher than actual need.

After manually right-sizing — auditing real usage patterns and adjusting resource requests to match — the team **cut their EKS bill by roughly 40%**.

---

### 3. Why Karpenter Doesn't Automatically Fix the Problem

Karpenter is the next-generation autoscaler for EKS, replacing the traditional Cluster Autoscaler. Karpenter calls the EC2 API directly, picks the optimal instance type for a pending pod, and can have a new node ready in as little as 45-60 seconds.

There's an important design limitation many teams overlook, though: **Karpenter fully trusts whatever resource requests you declare on a pod**. If a pod requests 4 vCPU but only actually uses 0.4 vCPU, Karpenter will still provision a node sized for 4 vCPU. Karpenter optimizes *provisioning*, not *declaration*.

---

### 4. The 7-Step Roadmap to Cutting EKS Costs

Below is the recommended execution order from Kubernetes cost optimization practitioners:

1. **Right-size pod resource requests first:** Use CloudWatch Container Insights to measure real CPU/memory usage, then adjust requests to match.
2. **Deploy Karpenter for dynamic node provisioning:** Replace rigid Auto Scaling Groups.
3. **Enable Spot Instances:** Use Spot for non-production environments, stateless services, or queue-based jobs (up to 90% savings).
4. **Migrate to Graviton (ARM) architecture:** Graviton instances are typically significantly cheaper than equivalent x86 instances.
5. **Add VPC endpoints:** Use VPC endpoints for frequently-called AWS services (S3, ECR) to eliminate NAT Gateway data transfer charges.
6. **Optimize EBS volumes:** Migrate from `gp2` to `gp3` and delete orphaned volumes.
7. **Consolidate Application Load Balancers:** Use a shared Ingress Controller instead of one ALB per service.

---

### 5. Comparing EKS Compute Purchasing Models

| Purchase Option | Savings vs. On-Demand | Best For | Key Risk |
| :--- | :--- | :--- | :--- |
| **On-Demand** | 0% (list price) | Experimental workloads, unstable traffic | Highest cost, no commitment |
| **Spot Instances** | Up to 90% | Queue jobs, batch processing, stateless services | Can be interrupted at any time |
| **Compute Savings Plans** | Typically 30-50% | Stable baseline load across EC2/Fargate/Lambda | 1-3 year commitment required |
| **Reserved Instances** | Comparable to Savings Plans | Workloads locked to a fixed instance type | Less flexible than Savings Plans |

---

### 6. EKS vs. GKE: A Cost Perspective

| Category | Amazon EKS | Google GKE |
| :--- | :--- | :--- |
| **Control Plane Fee** | ~$0.10/hour (~$73/month) | Zonal clusters are free; Regional clusters ~$0.10/hour |
| **On-demand Node** | Competitive, slightly cheaper for some families | Comparable, depends on instance type |
| **Discounted Capacity** | Spot Instances, up to 90% off | Preemptible/Spot VMs, up to 80% off |
| **Automation Mode** | EKS Auto Mode (separate management fee) | GKE Autopilot (billed by resource request) |

---

### 7. What the Savings Actually Look Like at Scale

Consider a reference example: a cluster running about **$85,000/month**. After completing the full 7-step roadmap above, estimated monthly savings land in the **$40,000-$55,000 range** — nearly half of the original bill.

---

### 8. Conclusion: Optimization Is a Process, Not a One-Time Fix

EKS costs don't balloon overnight — they creep up over time as resource requests get declared 'just to be safe.' Measure first, automate provisioning, use cheap capacity for the right workloads, and only then commit long-term.

