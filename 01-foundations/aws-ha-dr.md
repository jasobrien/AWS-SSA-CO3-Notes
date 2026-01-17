# High Availability & Disaster Recovery on AWS (SAA-C03) — Detailed Notes

This document focuses on availability concepts (Multi-AZ) and disaster recovery (Multi-Region) patterns that frequently appear in SAA-C03 scenarios.

---

## Quick Decision Guide

- **Need availability within a region?** → **Multi-AZ design** (multiple AZs, load balancing, autoscaling, Multi-AZ databases)
- **Need disaster recovery across regions?** → **Multi-Region** (replication + failover)
- **Need the lowest RTO/RPO?** → **Active-active (multi-region) + automated failover**
- **Need a low-cost DR approach with slower recovery?** → **Backup & restore**
- **Need DNS-based failover?** → **Route 53 health checks + failover routing**

---

## Foundations Summary (read this first)

- **High Availability (HA)** is usually **Multi-AZ** (within one region) and is the default expectation for “production”.
- **Disaster Recovery (DR)** is **Multi-Region** (a region is down) and is chosen based on **RTO/RPO**.
- Cost typically increases as you move: **backup/restore → pilot light → warm standby → active-active**.
- Exam wording cues:
	- “Must survive AZ failure” → **Multi-AZ**
	- “Region-wide disaster / global users / fastest recovery” → **Multi-Region**
- Pair this with deployment rollbacks/strategies in [aws-deployment-iac.md](aws-deployment-iac.md).

---

## 1. Key Concepts: RTO and RPO

- **RTO (Recovery Time Objective)**: how long you can be down.
- **RPO (Recovery Point Objective)**: how much data loss you can tolerate.

Rule of thumb:

- Lower RTO/RPO usually costs more (more duplication, more automation).

---

## 2. High Availability (Within a Region)

### Multi-AZ compute

- Deploy across **at least two AZs**.
- Use an **ALB/NLB** with targets in multiple AZs.
- Use **Auto Scaling Groups** or managed services (ECS/EKS) spanning AZs.

### Multi-AZ databases

- **RDS Multi-AZ**: synchronous standby, automatic failover.
- **Aurora**: distributed storage and fast failover patterns.

Exam “tell”:

- “**Must survive an AZ failure**” → Multi-AZ

---

## 3. DR Strategies (Cross-Region)

These are the classic patterns (ordered from cheapest/slowest to most expensive/fastest):

### 3.1 Backup and Restore

- Take backups/snapshots and restore in another region when needed.

Pros:

- Lowest cost

Cons:

- Highest RTO/RPO (restore takes time; data loss depends on backup frequency)

Use when:

- Cost-sensitive, relaxed recovery requirements

### 3.2 Pilot Light

- Keep a minimal version of core services running in DR region.
- Store data backups/replication there.

Pros:

- Better RTO than backup/restore

Cons:

- Still requires scaling up during DR

Use when:

- You need a smaller “always-on core” in DR region

### 3.3 Warm Standby

- A scaled-down but fully functional environment always running.
- Scale up when disaster occurs.

Pros:

- Lower RTO than pilot light

Cons:

- Ongoing cost (running resources)

### 3.4 Active-Active (Multi-Site)

- Full production traffic served from multiple regions.

Pros:

- Lowest RTO and very low RPO (depending on replication strategy)

Cons:

- Highest complexity and cost

Exam “tell”:

- “**Near-zero downtime requirement**” → active-active

---

## 4. Data Replication Options (Common Exam Points)

### S3

- **CRR (Cross-Region Replication)** for objects.

### DynamoDB

- **Global Tables** for multi-region active-active replication.

### RDS

- **Cross-region read replicas** (engine-dependent) for DR/read scaling.

### Aurora

- **Aurora Global Database** for low-latency cross-region replication.

Exam “tell”:

- “**Multi-region active-active NoSQL**” → DynamoDB Global Tables
- “**Multi-region relational with fast cross-region replication**” → Aurora Global Database

---

## 5. Failover and Traffic Switching

### Route 53

- **Failover routing** with health checks.
- **Weighted routing** for gradual cutover and testing.

### Global Accelerator vs Route 53 (short distinction)

- **Route 53**: DNS-based routing
- **Global Accelerator**: uses AWS global network for improved latency/failover for TCP/UDP applications

---

## 6. Common Pitfalls

- **Single-AZ deployments**: instance in one AZ, DB in one AZ, no redundancy.
- **NAT Gateway as a single point of failure**: design one per AZ for AZ resilience.
- **Not testing restore/failover**: backups are only as good as the restore process.
- **Overlapping CIDRs**: blocks future connectivity for hybrid/multi-VPC designs.

---

## 7. SAA-C03 “Tell” Phrases to Watch For

- “**Survive an Availability Zone outage**” → Multi-AZ design
- “**Disaster recovery across regions**” → Multi-Region pattern
- “**Cheapest DR**” → backup and restore
- “**Low RTO, moderate cost**” → warm standby
- “**Near-zero downtime**” → active-active
- “**Replicate S3 objects cross-region**” → S3 CRR
- “**Active-active multi-region NoSQL**” → DynamoDB Global Tables
- “**Fast cross-region relational replication**” → Aurora Global Database
