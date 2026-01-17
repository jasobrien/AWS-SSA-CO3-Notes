# AWS Solutions Architect – Associate (SAA‑C03)
## Final 90‑Minute Exam Cram (Only What Scores Marks)

---

## How to Use This (90 Minutes Total)
- **60 mins**: Read cover to cover
- **20 mins**: Re‑read *Exam Red Flags* only
- **10 mins**: Say answers out loud: *“If the question says X → choose Y”*

If it’s not in this document, it’s unlikely to score you marks.

---

## 1. Compute & Load Balancing (Highest ROI)

### What Scores Marks
- **Amazon EC2 (Elastic Compute Cloud)** runs virtual machines with full OS control.
- **Auto Scaling Group (ASG)** automatically scales EC2 instances across **multiple Availability Zones (AZs)**.
- **Application Load Balancer (ALB)** operates at **Layer 7 (HTTP/HTTPS)** and supports **path‑based and host‑based routing**.
- **Network Load Balancer (NLB)** operates at **Layer 4 (TCP/UDP)** and supports **very high throughput and static IPs**.
- **Spot Instances** are the **cheapest** EC2 pricing model but **can be interrupted**.
- **AWS Lambda** is **serverless**, **stateless**, **event‑driven**, and **short‑running**.

### Question Patterns to Recognise
- “**Highly available web application**” → **ALB + ASG across AZs**
- “**Path‑based routing**” → **ALB**
- “**Extreme performance / millions of requests**” → **NLB**
- “**Lowest cost batch processing**” → **Spot Instances**
- “**No servers to manage**” → **Lambda**

### Exam Red Flags 🚩
- Multi‑AZ **≠** Multi‑Region
- Lambda **≠** long‑running or stateful jobs
- NLB **does not understand HTTP paths**

---

## 2. Storage Selection (Guaranteed Marks)

### What Scores Marks
- **Amazon S3 (Simple Storage Service)** is **object storage** with **11 nines durability**.
- **S3 Storage Classes** trade cost vs access frequency (Standard, Infrequent Access, Glacier).
- **Amazon EBS (Elastic Block Store)** is **block storage**, attached to **one EC2 in one AZ**.
- **Amazon EFS (Elastic File System)** is a **shared file system** across multiple EC2 instances.
- **Lifecycle Policies** automatically move data to cheaper tiers.

### Question Patterns to Recognise
- “**Static website hosting**” → **S3**
- “**Shared files across EC2**” → **EFS**
- “**Attach disk to EC2**” → **EBS**
- “**Lowest cost archival storage**” → **S3 Glacier**

### Exam Red Flags 🚩
- S3 **is not a file system**
- EBS **cannot span AZs**
- Replication **is not backup**

---

## 3. Databases (Availability vs Scaling Traps)

### What Scores Marks
- **Amazon RDS (Relational Database Service) Multi‑AZ** = **automatic failover**, not scaling.
- **Read Replicas** = **read scalability**, not failover.
- **Amazon Aurora** provides **higher availability** and **faster replication** than standard RDS.
- **Amazon DynamoDB** is **serverless NoSQL** with automatic scaling.
- **Amazon ElastiCache** improves performance by **caching reads**.

### Question Patterns to Recognise
- “**Automatic failover**” → **RDS Multi‑AZ or Aurora**
- “**Scale reads**” → **Read Replicas**
- “**Unpredictable traffic, key‑value data**” → **DynamoDB**
- “**Reduce database load**” → **ElastiCache**

### Exam Red Flags 🚩
- Read Replicas **do not handle writes**
- DynamoDB is **eventually consistent by default**
- Redshift **is not for OLTP workloads**

---

## 4. VPC & Networking (Common Trick Questions)

### What Scores Marks
- **Amazon VPC (Virtual Private Cloud)** is an isolated network.
- **Security Groups** are **stateful** and apply at the **instance level**.
- **Network ACLs (Access Control Lists)** are **stateless** and apply at the **subnet level**.
- **Internet Gateway (IGW)** enables inbound and outbound internet access.
- **NAT Gateway** allows **outbound internet only** from private subnets.
- **Amazon Route 53** provides DNS‑based routing.

### Question Patterns to Recognise
- “**Private subnet needs internet access**” → **NAT Gateway**
- “**Explicit deny rules required**” → **Network ACL**
- “**DNS‑based failover**” → **Route 53**
- “**Low‑latency global users**” → **CloudFront**

### Exam Red Flags 🚩
- NAT Gateway **does not allow inbound traffic**
- Security Groups **do not support deny rules**
- CloudFront **does not host applications**

---

## 5. Security & IAM (Easy Marks)

### What Scores Marks
- **AWS IAM (Identity and Access Management) Roles** are preferred over access keys.
- **AWS KMS (Key Management Service)** manages encryption keys, not data.
- **AWS Secrets Manager** securely stores and rotates credentials.
- **AWS WAF (Web Application Firewall)** protects **HTTP/HTTPS (Layer 7)**.
- **AWS Shield** protects against **Distributed Denial of Service (DDoS)** attacks.

### Question Patterns to Recognise
- “**EC2 accessing AWS services securely**” → **IAM Role**
- “**Encrypt data at rest**” → **KMS**
- “**Protect web app from SQL injection**” → **WAF**
- “**DDoS protection**” → **Shield**

### Exam Red Flags 🚩
- KMS **does not store data**
- IAM Users **should not be used by applications**
- WAF **is not network‑layer protection**

---

## 6. Messaging, Decoupling & Monitoring

### What Scores Marks
- **Amazon SQS (Simple Queue Service)** decouples application components.
- **Amazon SNS (Simple Notification Service)** pushes messages to subscribers.
- **SNS pushes, SQS polls**.
- **Amazon CloudWatch** monitors metrics, logs, and alarms.
- **AWS CloudTrail** records API calls for auditing.

### Question Patterns to Recognise
- “**Decouple application tiers**” → **SQS**
- “**Fan‑out notifications**” → **SNS**
- “**Track API activity**” → **CloudTrail**
- “**Monitor CPU or set alarms**” → **CloudWatch**

### Exam Red Flags 🚩
- Queues **do not guarantee exactly‑once delivery**
- CloudTrail **is not a performance monitoring tool**
- SNS **is not a queue**

---

## Final Exam Brain Reset (Read Last)

If stuck between answers, choose the option that is:
1. **Managed**
2. **Highly available (Multi‑AZ)**
3. **Cost‑optimised**
4. **Least operational effort**

AWS almost never wants:
- Manual servers
- Custom security solutions
- Single‑AZ designs

Trust the patterns. You’re ready.

