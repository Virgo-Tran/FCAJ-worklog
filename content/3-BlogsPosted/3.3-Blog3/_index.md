---
title: "Blog 3: AWS Lambda Durable Functions"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# AWS Lambda Durable Functions: Building Reliable Workflows Without Step Functions

> **Category:** *Next-Generation Serverless Workflows*  
> **Author:** Tran Quoc Khanh  
> **Community:** [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj)  

*The newest capability from re:Invent 2025 lets Lambda break past its 15-minute limit — here's how it works, and when to actually use it.*

---

If you've ever written a Lambda function that needed to wait for human approval, wait hours for a webhook response, or orchestrate a multi-step AI pipeline running across an entire day, you've felt Lambda's 15-minute execution limit firsthand. The traditional answer has always been AWS Step Functions — but that means learning Amazon States Language (ASL), pulling your logic out of your familiar codebase, and managing a separate orchestration layer.

At **re:Invent 2025**, AWS introduced **Lambda Durable Functions** — a fundamentally different approach: build multi-step workflows directly inside Lambda, using the programming language you already know, with no ASL required.

---

### 1. What Are Lambda Durable Functions?

At its core, a Durable Function is still an ordinary Lambda function — same event handler, same integrations you already use. The difference is you add the open-source **Durable Execution SDK** (supporting JavaScript/TypeScript, Python, and Java) to your code, giving you a `DurableContext` with special operations:
* `step()`: A unit of work.
* `wait()`: A time-based pause.
* `callback()`: Waiting for an external signal.
* `parallel()` / `map()`: Concurrent execution.
* Child contexts: Sub-workflows.

Thanks to this model, a single durable execution can span anywhere from **a few minutes up to a full year**, across many separate Lambda invocations, **without incurring compute charges while waiting**.

---

### 2. How Checkpoint and Replay Actually Work

The core technical mechanic runs through four stages:

1. **Start:** A durable execution begins when you invoke a Lambda function with durable execution enabled.
2. **Checkpoint:** Every time your code hits a durable operation like `step()`, the SDK executes it, waits for the result, and persists (checkpoints) that result into an AWS-managed journal before moving on.
3. **Suspend & Resume:** When the code hits a wait or callback, the current Lambda invocation ends completely (no compute charges accrue), and a new invocation is scheduled or triggered once the condition is satisfied.
4. **Replay:** When the new invocation starts, the SDK re-runs the entire handler from the top. For completed steps, it loads stored results from the journal instead of re-executing — fast-forwarding in milliseconds before continuing from where it left off.

> **Hard Requirement — Determinism:**  
> Because the mechanism relies on replaying your handler, any non-deterministic code (generating random UUIDs, reading current timestamps, calling external APIs) must be wrapped inside a `step()` operation.

---

### 3. Limitations You Need to Know Before You Start

* **Creation time only:** Durable execution can only be enabled when creating a new Lambda function; you cannot convert an existing function.
* **Requires dedicated IAM permissions:** `lambda:CheckpointDurableExecution` and `lambda:GetDurableExecutionState`.
* **Regional availability:** Available in 15 AWS regions across US, Europe, and Asia-Pacific.

---

### 4. Head-to-Head: Durable Functions vs. Step Functions

| Criteria | Lambda Durable Functions | AWS Step Functions |
| :--- | :--- | :--- |
| **Workflow Definition** | Ordinary code (JS/TS, Python, Java) | Amazon States Language (JSON/YAML) |
| **AWS Integration** | Mostly Lambda-to-Lambda | Native built-in integrations (SQS, DynamoDB, ECS) |
| **Observability** | Via CLI/SDK; no visual builder yet | Visual workflow builder, easy step-by-step tracing |
| **Pricing Model** | $8/million operations + Lambda compute | Billed per state transition |
| **Maximum Wait Time** | Up to 1 year (no compute charge while waiting) | Unlimited (Standard Workflow, storage fees apply) |
| **Best Suited For** | Workflows that are mostly Lambda | Cross-service orchestration across teams |

---

### 5. The Cost Math: Up to 4x Cheaper

Durable Functions bill across three dimensions: $8 per million durable operations, checkpoint data storage, and standard Lambda compute. In contrast, Step Functions Standard bills ~$0.000025 per state transition. 

For typical approval workflows, real-world cost breakdowns show **Durable Functions can come out around 4x cheaper than Step Functions Standard**.

---

### 6. When Should You Choose Which?

* **Choose Lambda Durable Functions when:**
  - Your workflow is mostly Lambda calling Lambda with pauses in between.
  - You want to keep orchestration logic inside your application codebase.
  - Building multi-step AI pipelines without managing a separate orchestration layer.

* **Choose AWS Step Functions when:**
  - Directly orchestrating multiple AWS services (SQS -> DynamoDB -> ECS -> SNS) natively.
  - Requiring a visual workflow builder for cross-team visibility.
  - Heavy organizational investment in Step Functions.

---

### 7. Conclusion: The Hybrid Model

Many mature architectures use **Step Functions** for high-level cross-service orchestration, while leveraging **Durable Functions** inside individual Lambda branches for complex multi-step application logic.

Rule of thumb: If your workflow can be described as a single sequential function with a few pauses, try **Durable Functions**. If it's a multi-service orchestration diagram, **Step Functions** remains the sturdiest choice.

