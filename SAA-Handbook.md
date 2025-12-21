# AWS Solutions Architect – Associate (SAA-C03)

## Extended Edition – Detailed Service Reference & Exam-Relevant Attributes

This document expands every **exam-relevant AWS service** with **key attributes, limits, trade-offs, and how AWS tests them**. It is designed for **deep understanding**, not last‑minute cramming.

---

## 1. Compute Services

### Amazon EC2 (Elastic Compute Cloud)

**What it is:** Virtual servers with full operating system control.

**Key attributes to know:**

* **Instance families:**

  * General Purpose (t, m): balanced workloads
  * Compute Optimized (c): CPU-heavy
  * Memory Optimized (r, x): memory-heavy
* **Pricing models:**

  * On-Demand (no commitment)
  * Reserved Instances / Savings Plans (1–3 year commitment, cheaper)
  * Spot Instances (up to ~90% cheaper, interruptible)
* **Availability:** EC2 instances are **single-AZ**; high availability requires **multiple AZs + Auto Scaling Group**
* **Placement groups:**

  * Cluster (low latency)
  * Spread (fault isolation)

**Default limits & numbers (exam-expected):**

* High availability = **minimum 2 AZs**
* Stop/Start may change public IP

**Exam signals & traps:**

* "Highly available" never means single EC2
* Spot = cheapest, but **not guaranteed**

---

### Auto Scaling Groups (ASG)

**What it is:** Automatically adjusts EC2 capacity.

**Key attributes:**

* Scales based on CloudWatch metrics (CPU, ALB request count)
* Can span **multiple AZs**
* Supports health checks (EC2 + ELB)

**Exam signals:**

* Elastic workloads
* High availability without manual intervention

---

### Elastic Load Balancing (ELB)

#### Application Load Balancer (ALB)

* Layer 7 (HTTP/HTTPS)
* Path-based & host-based routing
* Supports WebSockets

#### Network Load Balancer (NLB)

* Layer 4 (TCP/UDP)
* Ultra-low latency
* Static IP addresses

**Exam signals:**

* HTTP routing → ALB
* Extreme performance / static IP → NLB

---

### AWS Lambda

**What it is:** Serverless event-driven compute.

**Key attributes:**

* Max execution time (short-running)
* Stateless
* Pay per execution
* Tight integration with S3, API Gateway, EventBridge

**Exam signals:**

* No server management
* Event-based processing

---

### Amazon ECS / EKS

#### ECS (Elastic Container Service)

* AWS-native container orchestration
* Simpler than Kubernetes

#### EKS (Elastic Kubernetes Service)

* Managed Kubernetes control plane
* More complex, more portable

**Exam signals:**

* Containers + AWS-native → ECS
* Kubernetes requirement → EKS

---

## 2. Storage Services (Fully Integrated – Exam Depth)

### Amazon S3 (Simple Storage Service)

**What it is:** Regional object storage designed for durability and scale.

**Key attributes:**

* Object-based (not block, not file)
* Storage classes: Standard, Standard-IA, One Zone-IA, Glacier Instant, Glacier Flexible, Glacier Deep Archive
* Lifecycle policies automate tiering
* Versioning protects against deletes/overwrites
* Server-side encryption (SSE-S3, SSE-KMS)

**Default limits & numbers (exam-expected):**

* **Durability:** 99.999999999% (11 nines)
* **Availability:** ~99.99% (class dependent)
* Objects up to **5 TB**

**Exam signals & traps:**

* Static website, backups, media → S3
* ❌ S3 is NOT a file system
* ❌ Replication ≠ backup

---

### Amazon EBS (Elastic Block Store)

**What it is:** Block storage volumes for EC2.

**Key attributes:**

* Attached to a single EC2
* Volume types: gp3 (general), io2 (high IOPS)
* Snapshots stored in S3

**Default limits & numbers:**

* **Single-AZ only**

**Exam signals & traps:**

* Boot volumes, databases on EC2 → EBS
* ❌ Cannot be shared across AZs

---

### Amazon EFS (Elastic File System)

**What it is:** Managed Network File System (NFS).

**Key attributes:**

* Shared across many EC2 instances
* Scales automatically

**Default limits & numbers:**

* **Multi-AZ by design**

**Exam signals & traps:**

* Shared files across EC2 → EFS
* ❌ More expensive than EBS

---

### Amazon FSx

**What it is:** Managed enterprise file systems.

**Key attributes:**

* FSx for Windows File Server
* FSx for Lustre (high-performance compute)

**Exam signals:**

* Windows-native file systems

---

### Amazon EBS (Elastic Block Store)

**What it is:** Block storage for EC2.

**Key attributes:**

* AZ-scoped
* Volume types: gp3, io1/io2
* Snapshots stored in S3

**Exam signals:**

* Boot volumes
* Databases on EC2

---

### Amazon EFS (Elastic File System)

**What it is:** Managed NFS file system.

**Key attributes:**

* Shared across EC2
* Scales automatically
* Multi-AZ by design

**Exam signals:**

* Shared file access

---

### Amazon FSx

* FSx for Windows File Server
* FSx for Lustre (high-performance)

---

### S3 Glacier

* Archive storage
* Retrieval times: minutes to hours

---

## 3. Database Services (High-Value, Trap-Heavy)

### Amazon RDS (Relational Database Service)

**What it is:** Managed relational databases (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server).

**Key attributes:**

* Automated backups and patching
* Vertical scaling
* Supports Multi-AZ and Read Replicas

**Default limits & numbers (exam-expected):**

* **Multi-AZ:** synchronous replication
* **Standby is NOT readable**

**Exam signals & traps:**

* "High availability relational DB" → RDS Multi-AZ
* ❌ Multi-AZ does NOT improve performance

---

### RDS Read Replicas

**What it is:** Asynchronous copies of RDS for read scaling.

**Key attributes:**

* Used only for reads
* Can be in same or different region

**Default limits & numbers:**

* **Asynchronous replication**

**Exam signals & traps:**

* "Scale reads" → Read Replicas
* ❌ Read Replicas do NOT provide failover

---

### Amazon Aurora

**What it is:** AWS-optimised relational database.

**Key attributes:**

* Distributed, shared storage
* Faster failover than RDS
* Compatible with MySQL/PostgreSQL

**Default limits & numbers:**

* **6 copies of data across 3 AZs**

**Exam signals & traps:**

* "Highest availability relational DB" → Aurora
* ❌ Aurora is NOT just tuned RDS

---

### Amazon DynamoDB

**What it is:** Serverless NoSQL key-value database.

**Key attributes:**

* Automatic scaling
* Partition key design is critical
* Global Tables for multi-region

**Default limits & numbers:**

* **Eventually consistent by default**

**Exam signals & traps:**

* "Unpredictable traffic" → DynamoDB
* ❌ Poor key design = hot partitions

---

### Amazon ElastiCache

**What it is:** In-memory data store.

**Key attributes:**

* Redis (persistence, replication)
* Memcached (simple caching)

**Exam signals & traps:**

* "Reduce DB load" → ElastiCache
* ❌ Cache is not source of truth

---

## 4. Networking & Content Delivery

### Amazon VPC (Virtual Private Cloud)

**What it is:** Isolated virtual network.

**Key attributes:**

* CIDR-based addressing
* Public and private subnets
* Route tables control traffic

**Default limits & numbers:**

* **Subnets are AZ-scoped**

**Exam signals & traps:**

* "Private network" → VPC
* ❌ One subnet ≠ high availability

---

### Internet Gateway (IGW)

**What it is:** Enables internet access for VPC.

**Exam signals & traps:**

* Public subnet requires IGW
* ❌ IGW alone does not make subnet public

---

### NAT Gateway

**What it is:** Allows outbound internet from private subnets.

**Default limits & numbers:**

* **Outbound only**

**Exam signals & traps:**

* "Private EC2 needs internet" → NAT Gateway
* ❌ NAT does not allow inbound traffic

---

### Security Groups vs Network ACLs

**Security Groups:**

* Stateful
* Instance-level
* Allow rules only

**Network ACLs:**

* Stateless
* Subnet-level
* Explicit allow and deny

**Exam traps:**

* ❌ Security Groups cannot deny traffic

---

### Amazon Route 53

**What it is:** Managed DNS service.

**Key attributes:**

* Routing policies: simple, weighted, latency, failover

**Exam signals:**

* "DNS failover" → Route 53

---

### Amazon CloudFront

**What it is:** Content Delivery Network (CDN).

**Key attributes:**

* Caches content at edge locations

**Exam traps:**

* ❌ CloudFront does not host apps

---

## 5. Security, Identity & Compliance

### AWS IAM (Identity and Access Management)

**What it is:** Controls access to AWS resources.

**Key attributes:**

* Users, Groups, Roles
* JSON policies

**Default limits & numbers:**

* **Roles preferred over access keys**

**Exam traps:**

* ❌ Applications should not use IAM users

---

### AWS KMS (Key Management Service)

**What it is:** Manages encryption keys.

**Exam traps:**

* ❌ KMS does not store data

---

### AWS Secrets Manager

**What it is:** Secure secret storage and rotation.

**Exam signals:**

* "Rotate credentials" → Secrets Manager

---

### AWS WAF & Shield

**WAF:**

* Layer 7 protection (HTTP/HTTPS)

**Shield:**

* DDoS protection

---

## 6. Messaging & Application Integration

### Amazon SQS (Simple Queue Service)

**What it is:** Managed message queue.

**Key attributes:**

* Standard vs FIFO

**Default limits & numbers:**

* **At-least-once delivery**

**Exam traps:**

* ❌ No exactly-once guarantee

---

### Amazon SNS (Simple Notification Service)

**What it is:** Pub/Sub messaging.

**Exam signals:**

* Fan-out patterns

---

### Amazon EventBridge

**What it is:** Event bus for services.

**Exam traps:**

* ❌ Not a queue

---

### AWS Step Functions

**What it is:** Serverless workflow orchestration.

**Exam signals:**

* Orchestrating Lambda workflows

---

## 7. Monitoring & Management

### Amazon CloudWatch

**What it is:** Monitoring and logging.

**Exam traps:**

* ❌ CloudWatch is not auditing

---

### AWS CloudTrail

**What it is:** API activity logging.

**Exam traps:**

* ❌ CloudTrail is not performance monitoring

---

### AWS Config

**What it is:** Configuration tracking.

---

### AWS Systems Manager

**What it is:** Operational management for EC2.

---

## Final Exam Guidance (Read Before Sitting Exam)

AWS almost always wants solutions that are:

* Managed
* Multi-AZ
* Loosely coupled
* Cost-optimised

Avoid:

* Single-AZ designs
* Custom-built solutions
* Long-running servers where serverless fits

If stuck, choose the option with **least operational effort**.

---

## Numbers-Only Last Page (Must Memorise)

### Global & Availability

* High availability = **2+ Availability Zones**
* Multi-AZ ≠ Multi-Region

### Compute

* **AWS Lambda max execution:** **15 minutes**
* **Elastic Load Balancer layers:**

  * ALB = **Layer 7**
  * NLB = **Layer 4**

### Storage

* **Amazon S3 durability:** **99.999999999% (11 nines)**
* **Amazon S3 max object size:** **5 TB**
* **Amazon EBS:** **Single-AZ only**
* **Amazon EFS:** **Multi-AZ by design**

### Databases

* **RDS Multi-AZ:** Failover only (no reads)
* **Read Replicas:** Read scaling only (async)
* **Amazon Aurora:** **6 copies across 3 AZs**
* **DynamoDB consistency:** Eventual by default

### Networking & Security

* **Security Groups:** Stateful, allow only
* **Network ACLs:** Stateless, allow + deny
* **NAT Gateway:** Outbound internet only
* **AWS WAF:** Layer 7 only

### Messaging

* **Amazon SQS:** At-least-once delivery
* **Amazon SNS:** Push-based messaging

### Exam Mindset

* Prefer **managed services**
* Prefer **Multi-AZ**
* Prefer **decoupling**
* Eliminate answers requiring manual ops
