# AWS Monitoring & Observability (SAA-C03) — Detailed Notes

This document covers CloudWatch, CloudTrail, X-Ray, and common “how do we monitor/alert/audit?” patterns.

---

## Quick Decision Guide

- **Need metrics, alarms, dashboards?** → **CloudWatch**
- **Need log collection and log queries?** → **CloudWatch Logs** (or OpenSearch for advanced search)
- **Need auditing of AWS API calls (who did what)?** → **CloudTrail**
- **Need distributed tracing across microservices?** → **X-Ray**
- **Need config compliance over time?** → **AWS Config**

---

## 1. Amazon CloudWatch

### 1.1 Metrics

- Service metrics (EC2 CPU, ALB request count, Lambda invocations, etc.)
- **Custom metrics** for app-specific signals

EC2 note (common exam point):

- Memory/disk are not default EC2 metrics; you typically need the CloudWatch Agent.

### 1.2 Alarms

- Trigger notifications/actions when a metric crosses a threshold.
- Can notify **SNS**, trigger Auto Scaling, or run actions.

Alarm design tips:

- Use the right statistic (Average vs p95/p99 if available).
- Choose an appropriate evaluation period to avoid flapping.

### 1.3 Dashboards

- Visualize metrics across services.

---

## 2. CloudWatch Logs

### Log ingestion

Common sources:

- EC2/On-prem: CloudWatch Agent
- Lambda: automatic log streams
- VPC Flow Logs: can publish to CloudWatch Logs

### Log insights

- **CloudWatch Logs Insights** is used to query logs.

Exam “tell”:

- “**Search and query logs quickly**” → Logs Insights

---

## 3. CloudWatch Events vs EventBridge (Context)

Historically CloudWatch Events evolved into EventBridge.

- For event-driven routing, **EventBridge** is usually the answer.

---

## 4. AWS CloudTrail (Audit)

### What it is

Records AWS API calls and events.

What it answers:

- “Who created/deleted/modified this resource?”
- “Which IAM principal called this API?”

Delivery:

- S3 and/or CloudWatch Logs

Exam “tell”:

- “**Investigate a security incident / audit activity**” → CloudTrail

---

## 5. AWS X-Ray (Tracing)

### What it is

Distributed tracing to see request flow through services.

Use when:

- Microservices performance troubleshooting
- Finding latency hotspots and downstream errors

Exam “tell”:

- “**Trace a request across services**” → X-Ray

---

## 6. AWS Config (Compliance over time)

### What it is

Tracks resource configurations and evaluates them against rules.

Use when:

- You need to know “what changed” and whether resources remain compliant

Exam “tell”:

- “**Detect configuration drift / enforce compliance**” → Config

---

## 7. Common Observability Patterns

### Alarming on application health

- ALB target health + CloudWatch alarms
- Synthetic canaries for endpoint checks (if mentioned)

### Centralized logging

- Send logs from multiple accounts/apps to a log account (common enterprise pattern).

### Audit + detective controls

- CloudTrail + Config + GuardDuty + Security Hub (security-focused view)

---

## 8. SAA-C03 “Tell” Phrases to Watch For

- “**Alert when CPU/network is high**” → CloudWatch alarms
- “**Need memory/disk utilization on EC2**” → CloudWatch Agent
- “**Query logs**” → CloudWatch Logs Insights
- “**Audit API calls / who changed what**” → CloudTrail
- “**Trace requests end-to-end**” → X-Ray
- “**Configuration drift / compliance reports**” → AWS Config
