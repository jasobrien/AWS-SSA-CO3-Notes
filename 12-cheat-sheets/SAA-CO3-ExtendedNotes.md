# AWS Solutions Architect – Associate (SAA-C03)
## Extended Edition – Detailed Service Reference & Exam-Relevant Attributes

This document expands every **exam-relevant AWS service** with **key attributes, limits, trade-offs, and how AWS tests them**. It is designed for **deep understanding**, not last‑minute cramming.

---

## 1. Compute Services

### Amazon EC2 (Elastic Compute Cloud)
**What it is:** Virtual servers with full operating system control.

**Key attributes to know:**
- **Instance families:**
  - General Purpose (t, m): balanced workloads
  - Compute Optimized (c): CPU-heavy
  - Memory Optimized (r, x): memory-heavy
- **Pricing models:**
  - On-Demand (no commitment)
  - Reserved Instances / Savings Plans (1–3 year commitment, cheaper)
  - Spot Instances (up to ~90% cheaper, interruptible)
- **Availability:** EC2 instances are **single-AZ**; high availability requires **multiple AZs + Auto Scaling Group**
- **Placement groups:**
  - Cluster (low latency)
  - Spread (fault isolation)

**Default limits & numbers (exam-expected):**
- High availability = **minimum 2 AZs**
- Stop/Start may change public IP

**Exam signals & traps:**
- "Highly available" never means single EC2
- Spot = cheapest, but **not guaranteed**

---

### Auto Scaling Groups (ASG)
**What it is:** Automatically adjusts EC2 capacity.

**Key attributes:**
- Scales based on CloudWatch metrics (CPU, ALB request count)
- Can span **multiple AZs**
- Supports health checks (EC2 + ELB)

**Exam signals:**
- Elastic workloads
- High availability without manual intervention

---

### Elastic Load Balancing (ELB)

#### Application Load Balancer (ALB)
- Layer 7 (HTTP/HTTPS)
- Path-based & host-based routing
- Supports WebSockets

#### Network Load Balancer (NLB)
- Layer 4 (TCP/UDP)
- Ultra-low latency
- Static IP addresses

**Exam signals:**
- HTTP routing → ALB
- Extreme performance / static IP → NLB

---

### AWS Lambda
**What it is:** Serverless event-driven compute.

**Key attributes:**
- Max execution time (short-running)
- Stateless
- Pay per execution
- Tight integration with S3, API Gateway, EventBridge

**Exam signals:**
- No server management
- Event-based processing

---

### Amazon ECS / EKS

#### ECS (Elastic Container Service)
- AWS-native container orchestration
- Simpler than Kubernetes

#### EKS (Elastic Kubernetes Service)
- Managed Kubernetes control plane
- More complex, more portable

**Exam signals:**
- Containers + AWS-native → ECS
- Kubernetes requirement → EKS

---

## 2. Storage Services

### Amazon S3 (Simple Storage Service)
**What it is:** Object storage with 11 nines durability.

**Key attributes:**
- Storage classes: Standard, IA, One Zone-IA, Glacier
- Lifecycle rules
- Versioning
- Server-side encryption
- Cross-region replication

**Exam signals:**
- Static content
- Backup
- Durable object storage

---

### Amazon EBS (Elastic Block Store)
**What it is:** Block storage for EC2.

**Key attributes:**
- AZ-scoped
- Volume types: gp3, io1/io2
- Snapshots stored in S3

**Exam signals:**
- Boot volumes
- Databases on EC2

---

### Amazon EFS (Elastic File System)
**What it is:** Managed NFS file system.

**Key attributes:**
- Shared across EC2
- Scales automatically
- Multi-AZ by design

**Exam signals:**
- Shared file access

---

### Amazon FSx
- FSx for Windows File Server
- FSx for Lustre (high-performance)

---

### S3 Glacier
- Archive storage
- Retrieval times: minutes to hours

---

## 3. Database Services

### Amazon RDS
**Key attributes:**
- Managed SQL (MySQL, Postgres, Oracle, SQL Server)
- Multi-AZ = failover
- Read Replicas = read scaling

---

### Amazon Aurora
- Distributed storage
- Faster failover
- Higher availability than RDS

---

### Amazon DynamoDB
- Serverless NoSQL
- Partition key critical
- On-demand vs provisioned capacity

---

### Amazon ElastiCache
- Redis (persistence, replication)
- Memcached (simple cache)

---

### Amazon Redshift
- Columnar data warehouse
- Analytics, not OLTP

---

## 4. Networking & Content Delivery

### Amazon VPC
- Subnets (public / private)
- Route tables
- Internet Gateway
- NAT Gateway

---

### Security Groups vs Network ACLs
- Security Groups: stateful, instance-level
- NACLs: stateless, subnet-level

---

### Amazon Route 53
- DNS
- Routing policies: simple, weighted, latency, failover

---

### Amazon CloudFront
- CDN
- Caches content globally

---

## 5. Security, Identity & Compliance

### AWS IAM
- Users, Groups, Roles
- Policies (JSON)

---

### AWS KMS
- Manages encryption keys

---

### AWS Secrets Manager
- Stores & rotates secrets

---

### Amazon Cognito
- User authentication

---

### AWS WAF & Shield
- WAF: Layer 7 protection
- Shield: DDoS protection

---

## 6. Monitoring & Management

### Amazon CloudWatch
- Metrics, logs, alarms

---

### AWS CloudTrail
- API auditing

---

### AWS Config
- Configuration history

---

### AWS Systems Manager
- Patch & manage EC2

---

## 7. Application Integration

### Amazon SQS
- Message queue

---

### Amazon SNS
- Pub/Sub messaging

---

### Amazon EventBridge
- Event bus

---

### AWS Step Functions
- Workflow orchestration

---

## 8. Service Comparison Tables (High-Value for Exam)

### Load Balancing Comparison

| Feature | ALB (Application Load Balancer) | NLB (Network Load Balancer) | CLB (Classic Load Balancer) |
|------|-------------------------------|-----------------------------|-----------------------------|
| OSI Layer | Layer 7 (HTTP/HTTPS) | Layer 4 (TCP/UDP) | Layer 4 & 7 (legacy) |
| Routing | Path & host based | No routing logic | Limited |
| Performance | High | Extreme, ultra-low latency | Moderate |
| Static IP | No | Yes | No |
| Exam Use | Web apps, APIs | High throughput, TCP | Rarely correct |

### Storage Comparison

| Feature | S3 | EBS | EFS |
|------|----|-----|-----|
| Storage Type | Object | Block | File |
| AZ Scope | Regional | Single AZ | Multi-AZ |
| Typical Use | Static files, backups | EC2 disks | Shared files |
| Scales Automatically | Yes | Manual | Yes |
| Exam Trap | Not a file system | AZ-bound | More expensive |

### Database Comparison (Core Exam Area)

| Feature | RDS Multi-AZ | Read Replica | Aurora | DynamoDB |
|------|-------------|--------------|--------|-----------|
| Purpose | High availability | Read scaling | HA + performance | Serverless NoSQL |
| Automatic Failover | Yes | No | Yes | Yes |
| Scaling Type | Vertical | Horizontal (reads) | Horizontal | Automatic |
| Consistency | Strong | Eventual | Strong | Eventual default |
| Exam Trap | Not for scaling | Not HA | Different engine | Key design critical |

### Messaging Comparison

| Feature | SQS | SNS | EventBridge |
|------|-----|-----|-------------|
| Model | Queue | Pub/Sub | Event bus |
| Delivery | Pull | Push | Push |
| Ordering | Optional (FIFO) | No | No |
| Typical Use | Decoupling | Fan-out | Event-driven apps |
| Exam Trap | No exactly-once | Not a queue | Not for buffering |

### Security Comparison

| Feature | Security Group | Network ACL | WAF |
|------|----------------|-------------|-----|
| Level | Instance | Subnet | Application |
| Stateful | Yes | No | Yes |
| Deny Rules | No | Yes | Yes |
| Layer | L3/L4 | L3/L4 | L7 |
| Exam Use | Default firewall | Explicit deny | HTTP protection |

---

## 9. Default Limits & Numbers AWS Expects You to Remember (Exam Gold)

These numbers appear implicitly in questions. You are not asked directly, but the *correct answer assumes you know them*.

### Compute & Load Balancing

- **Availability Zones (AZs):** High availability = **at least 2 AZs**
- **Auto Scaling Groups:** Scale across AZs by default
- **AWS Lambda:**
  - Max execution time: **15 minutes**
  - Stateless between invocations
- **Elastic Load Balancer:**
  - ALB = Layer **7**
  - NLB = Layer **4**

### Storage

- **Amazon S3 durability:** **99.999999999% (11 nines)**
- **Amazon S3 availability:** ~**99.99%** (varies by class)
- **Amazon EBS:**
  - Volumes are **single-AZ only**
  - Snapshots are stored in **S3** (regional)
- **Amazon EFS:**
  - **Multi-AZ by design**

### Databases

- **Amazon RDS Multi-AZ:**
  - Standby is **not readable**
- **Read Replicas:**
  - Used for **read scaling only**
  - Asynchronous replication
- **Amazon DynamoDB:**
  - Default consistency: **eventual**
  - Strong consistency must be explicitly requested

### Networking

- **Security Groups:**
  - Stateful
  - Allow rules only (implicit deny)
- **Network ACLs:**
  - Stateless
  - Explicit allow and deny rules
- **NAT Gateway:**
  - **Outbound-only** internet access

### Security

- **IAM Best Practice:**
  - Use **roles**, not long-term access keys
- **AWS KMS:**
  - Manages **keys**, not encrypted data
- **AWS WAF:**
  - Protects **HTTP/HTTPS (Layer 7)** only

### Messaging & Integration

- **Amazon SQS:**
  - At-least-once delivery (duplicates possible)
  - FIFO queues preserve order
- **Amazon SNS:**
  - Push-based messaging

---

## Final Exam Guidance

Choose architectures that are:
- Managed
- Multi-AZ
- Loosely coupled
- Cost-optimised

Avoid single points of failure and custom solutions unless explicitly required.

