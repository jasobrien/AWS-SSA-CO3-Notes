# AWS Well-Architected & Cost Optimization (SAA-C03) — Detailed Notes

This document summarizes AWS Well-Architected thinking and cost optimization levers that often determine the “best answer” in SAA-C03 scenarios.

---

## Quick Decision Guide

- **Need a structured way to evaluate architecture trade-offs?** → **Well-Architected Framework**
- **Need to reduce compute cost for steady workloads?** → **Savings Plans / Reserved Instances**
- **Need cheapest compute for fault-tolerant workloads?** → **Spot**
- **Need to reduce NAT Gateway spend to reach AWS services?** → **VPC Endpoints**
- **Need to reduce S3 cost for older data?** → **Lifecycle policies + appropriate storage classes**
- **Need to track spend by team/app?** → **Tags + Cost Allocation + Cost Explorer/Budgets**

---

## Foundations Summary (read this first)

- Well-Architected is the exam’s “tie-breaker”: choose options that improve **reliability/security** without unreasonable cost/ops burden.
- Most “reduce cost” answers come from a small set of levers:
	- **Commitment** (Savings Plans/RIs) for steady usage
	- **Spot** for interruptible work
	- **Right-sizing + Auto Scaling** for variable demand
	- **S3 lifecycle/storage class** for data aging
	- **VPC endpoints** to reduce **NAT Gateway** spend
- Governance is often the first step: tags + **Budgets** + Cost Explorer/CUR.
- Pair this with HA/DR trade-offs in [aws-ha-dr.md](aws-ha-dr.md).

---

## 1. Well-Architected Framework (WA)

Five pillars (commonly referenced):

- **Operational Excellence**
- **Security**
- **Reliability**
- **Performance Efficiency**
- **Cost Optimization**

Exam usage:

- The best answer often improves one pillar without severely harming others.

---

## 2. Cost Visibility and Governance

### Cost allocation tags

- Use tags like `CostCenter`, `App`, `Environment`, `Owner`.
- Enable them as cost allocation tags.

### Tools

- **Cost Explorer**: analyze historical spend
- **AWS Budgets**: alerts on forecast/actual costs
- **CUR (Cost and Usage Report)**: detailed billing data (analysis in Athena/Redshift)

Exam “tell”:

- “**Alert when monthly spend exceeds threshold**” → Budgets

---

## 3. Compute Cost Levers

### Commitment-based discounts

- **Savings Plans** (common recommendation for broad compute savings)
- **Reserved Instances** (service-specific; still tested)

### Spot

- Great for batch, stateless, fault-tolerant workloads.

### Rightsizing

- Choose appropriate instance size
- Use Auto Scaling for variable demand

Exam “tell”:

- “**Steady baseline usage, want discount**” → Savings Plans / RIs
- “**Interruptions OK**” → Spot

---

## 4. Storage Cost Levers

### S3 storage classes + lifecycle

- Use lifecycle policies to transition older objects.
- Use Intelligent-Tiering if access patterns are unknown.

### EBS

- Use appropriate volume type (gp3 vs io2 etc.).
- Delete unattached volumes and old snapshots when safe.

Exam “tell”:

- “**Data rarely accessed after 90 days**” → lifecycle to IA/Glacier

---

## 5. Data Transfer and Networking Cost

### NAT Gateway cost pitfalls

- NAT Gateway can be a major cost driver due to per-GB processing charges.

Cost optimization:

- Use **Gateway VPC Endpoint** for S3/DynamoDB.
- Use **Interface Endpoints** for AWS APIs (when traffic is large and private access is needed).

Exam “tell”:

- “**High NAT costs accessing S3 from private subnets**” → S3 Gateway Endpoint

### CloudFront

- Cache content closer to users → reduces origin load and data transfer.

---

## 6. Reliability vs Cost (Common Trade-offs)

- Multi-AZ improves availability but costs more than single-AZ.
- Multi-region DR adds cost and operational complexity.

Exam framing:

- If a question emphasizes “mission critical” and “must be highly available”, multi-AZ is usually correct even if it costs more.

---

## 7. SAA-C03 “Tell” Phrases to Watch For

- “**Reduce monthly AWS bill**” → rightsizing + Savings Plans/RIs + lifecycle policies
- “**Alert on cost spikes**” → AWS Budgets
- “**Unknown S3 access patterns**” → Intelligent-Tiering
- “**High NAT costs**” → VPC endpoints
- “**Fault-tolerant batch**” → Spot
- “**Need global caching to reduce origin load**” → CloudFront
