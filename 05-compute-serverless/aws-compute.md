# AWS Compute (SAA-C03) — Detailed Notes

This document is a compute-focused companion to [aws-webapp-architectures.md](../09-architectures/aws-webapp-architectures.md). It covers the major AWS compute choices, what they’re good at, and common SAA-C03 style decision points.

---

## Quick Decision Guide

Use this as a first-pass filter:

- **Need full OS control / custom agents / long-running processes?** → **EC2**
- **Need “run containers” with minimal server management?** → **ECS on Fargate** (or **EKS on Fargate** if you specifically need Kubernetes)
- **Need event-driven “run code on demand”** (spiky traffic, short tasks, many integrations)? → **Lambda**
- **Need managed Kubernetes** (multi-team platform, K8s ecosystem, standardization)? → **EKS**
- **Need simple PaaS for web apps** (quick deploy, minimal ops)? → **Elastic Beanstalk** or **App Runner**
- **Need a simple VPS-style experience**? → **Lightsail**
- **Need large-scale batch processing** (queues, retries, fleets)? → **AWS Batch**

---

## 1. EC2 (Elastic Compute Cloud)

### What it is

A virtual machine you manage (OS + patches + packages), running inside a VPC.

### Core building blocks

- **AMI**: template for your instance (OS + preinstalled software)
- **Instance type**: CPU/memory/network characteristics (e.g., general purpose vs memory optimized)
- **EBS volumes**: network-attached block storage
- **Security group**: instance-level stateful firewall
- **ENI (Elastic Network Interface)**: the network “card” for an instance
- **User data**: bootstrap script run at first boot (commonly via cloud-init)

### Storage choices

- **EBS-backed instances**: most common. Supports snapshots, encryption, persistent storage.
- **Instance store**: ephemeral local disk. Very fast, but data is lost on stop/terminate.

Typical exam mapping:

- “Need persistence / snapshots / encryption / backups” → **EBS**
- “Need super-fast scratch space; losing data is fine” → **Instance store**

### Networking basics

- Instances live in a **subnet** (public or private).
- **Public IPv4** (or **Elastic IP**) is required for direct inbound from the internet.
- **Private IPv4** is always present and used inside the VPC.
- **Security groups** are stateful; **NACLs** are stateless (subnet-level).

### Access & management

- **SSH** is possible, but for AWS best-practice ops (and exam answers), prefer:
  - **AWS Systems Manager (SSM) Session Manager** (no inbound ports required)
  - **AmazonSSMManagedInstanceCore** on the instance role

### Monitoring & health

- **EC2 status checks**
  - System status check: underlying host issues
  - Instance status check: OS/network issues inside the VM
- **CloudWatch metrics** (CPU is default; memory/disk need agent)
- **CloudWatch Logs** via the CloudWatch Agent

### Scaling

- Manual scaling: add/remove instances
- Managed scaling: **Auto Scaling Groups (ASG)** (see section 2)

### Pricing / purchasing options (high-yield)

- **On-Demand**: pay per use; flexible
- **Savings Plans / Reserved Instances (RIs)**: commit for discount (steady workloads)
- **Spot Instances**: deep discount, can be interrupted (fault-tolerant workloads)
- **Dedicated Hosts / Dedicated Instances**: tenancy/compliance needs

Exam “tell”:

- “Interruptions acceptable” → **Spot**
- “Steady baseline, want discount” → **RIs / Savings Plans**
- “License bound to physical host / compliance” → **Dedicated Host**

### Instance type costs (right level for exam)

On the SAA exam, you rarely need exact $/hour pricing. You *do* need the cost **directionally**:

- **General purpose (M)**: balanced CPU/memory → default choice; good cost for “typical web/app servers”
- **Burstable (T)**: cheapest for low/variable CPU with a baseline; uses **CPU credits**
  - Great for dev/test, small services, spiky-but-not-constant CPU
  - Trap: sustained CPU can run out of credits (performance drops) → pick M/C if CPU is consistently high
- **Compute optimized (C)**: more CPU per dollar when CPU-bound (batch, encoding, compute-heavy)
- **Memory optimized (R/X)**: more RAM → typically higher cost; pick when memory-bound (caches, large JVM heaps)
- **Accelerated (P/G/Inf)**: GPU/ML inference → most expensive; choose only when explicitly required

High-yield cost levers:

- **Graviton (ARM, e.g., `t4g`, `m7g`, `c7g`)**: often better price/performance than x86 *if your workload supports ARM*
- **Newer generation** is usually better price/perf than older generation (all else equal)
- **Right-size**: most cost waste is over-provisioning CPU/RAM; use metrics to pick size/family
- **Mix instance types + Spot** in ASG (Mixed Instances Policy) for cheaper capacity when interruption is acceptable

What “cost-effective EC2” typically means in questions:

- “Steady workload” → **Savings Plans / RIs**
- “Fault-tolerant / can be interrupted” → **Spot**
- “Low average CPU, occasional spikes” → **T family**
- “CPU pegged all day” → **C/M family** (not T)

EC2 “total cost” reminder (common exam trick):

- **Compute**: instance-hours (On-Demand vs RI/Savings Plan vs Spot)
- **Storage**: EBS volumes (size + type) and snapshots
- **Networking**: data transfer out to internet, inter-AZ traffic, and NAT Gateway processing (often a surprise big cost)
- **Add-ons**: load balancers, CloudWatch Logs/metrics, and other managed services you attach

### Security highlights

- Use **IAM roles** (instance profiles) instead of static access keys.
- If using instance metadata, prefer **IMDSv2** (session-oriented) over IMDSv1.
- Encrypt EBS volumes and snapshots.

---

## 2. EC2 Auto Scaling (ASG)

### What it is

Automatically adds/removes EC2 instances based on demand and health.

### Key concepts

- **Launch template**: instance configuration (AMI, instance type, security groups, user data)
- **Desired / Min / Max capacity**: target and bounds
- **Health checks**:
  - EC2 status checks
  - Optional: ELB target health (recommended for web apps)

### Scaling policy types

- **Target tracking**: “keep CPU around 50%” (common and simple)
- **Step scaling**: different actions based on alarm thresholds
- **Scheduled scaling**: known patterns (e.g., business hours)

### Common exam points

- Put instances across **multiple AZs** for HA.
- Use **ELB + ASG** together for web tiers.
- Use **Instance Refresh** for rolling updates of instances/AMIs.
- Use **Mixed Instances Policy** to diversify instance types (and add Spot).

---

## 3. Elastic Load Balancing (ELB)

### Which load balancer when?

#### Application Load Balancer (ALB)

- Layer 7 (HTTP/HTTPS)
- **Host/path-based routing** (e.g., `/api` → one target group, `/static` → another)
- Integrates well with **ECS**, **Lambda targets**, and **WAF**
- Supports **WebSockets**

Use ALB when:

- You need L7 routing rules, HTTP features, or multiple services behind one endpoint.

#### Network Load Balancer (NLB)

- Layer 4 (TCP/UDP/TLS)
- Handles very high throughput / low latency
- Supports **static IPs** and **Elastic IPs** per AZ
- Preserves source IP (common requirement)

Use NLB when:

- You need static IPs, extreme performance, or non-HTTP protocols.

#### Gateway Load Balancer (GWLB)

- Inserts network appliances (firewalls/IDS/IPS) transparently
- Operates with **GENEVE** encapsulation

Use GWLB when:

- You need to scale third-party network appliances in-line.

#### Classic Load Balancer (CLB)

- Legacy. ALB/NLB are preferred.

### Health checks (high-yield)

- Misconfigured health checks are a common root cause of “targets keep draining”.
- Prefer application-level health endpoints (e.g., `/health`).

---

## 4. Lambda (Serverless Functions)

### What it is

Run code in response to events without managing servers.

### Key characteristics

- **Event-driven**: S3 events, EventBridge, API Gateway, SQS, SNS, DynamoDB Streams, etc.
- **Time-limited execution**: designed for short-lived workloads
- **Stateless by default**: use external state (DynamoDB, S3, RDS, etc.)

### Performance & scaling concepts

- **Concurrency**: number of parallel executions
  - **Reserved concurrency**: cap or guarantee for a function
  - **Provisioned concurrency**: reduce cold starts
- **Cold start**: initialization latency when new environments are created

### VPC integration (common exam trap)

- If a Lambda function needs access to private resources (e.g., RDS in private subnet), it can run in a VPC.
- If it needs outbound internet while in a private subnet, you typically need a **NAT Gateway** (or suitable VPC endpoints depending on the destination).

### Packaging & deployments

- Zip package or container image
- **Versions and aliases** enable safe deployments (e.g., canary/linear)

### When Lambda is a great answer

- Spiky traffic, unpredictable load
- Many integrations and glue code
- Event processing pipelines

### When Lambda is _not_ a great answer

- Very long-running tasks
- Heavy, always-on compute with predictable usage (might be cheaper on containers/EC2)
- Workloads requiring deep OS customization

---

## 5. Containers on AWS

### 5.1 ECS (Elastic Container Service)

**Core concepts**

- **Cluster**: logical grouping of capacity
- **Task definition**: what to run (image, CPU/memory, env vars, IAM roles)
- **Service**: keeps desired count of tasks running, integrates with load balancers
- **ECR (Elastic Container Registry)**: container image storage

**Launch types**

- **Fargate**: serverless containers (no EC2 management)
- **EC2 launch type**: you manage EC2 worker instances

**IAM roles (high-yield)**

- **Task execution role**: pull image from ECR, write logs
- **Task role**: app permissions (S3, DynamoDB, SQS, Secrets Manager, etc.)

**Networking**

- Commonly uses **awsvpc mode** (task gets its own ENI + security group)

When ECS on Fargate is a great answer:

- You want containers without managing servers
- You need predictable networking/isolation per task

### 5.2 EKS (Elastic Kubernetes Service)

**What it is**
Managed Kubernetes control plane.

**Compute options**

- Managed node groups (EC2 workers)
- Fargate profiles (serverless pods for eligible workloads)

**Identity**

- Prefer IRSA (IAM Roles for Service Accounts) for pod-level AWS permissions.

When EKS is a great answer:

- You need Kubernetes compatibility, tooling, and ecosystem
- You’re standardizing a platform for many teams

Trade-offs:

- More operational complexity than ECS (even though control plane is managed)

---

## 6. Elastic Beanstalk (EB)

### What it is

Platform-as-a-Service for common web app stacks.

It typically provisions:

- An environment (single instance or load-balanced)
- Auto scaling
- A load balancer
- Deployment policies

Use EB when:

- You want quick deployment with AWS-managed scaffolding
- You can fit your app into supported platforms and don’t need deep infrastructure customization

---

## 7. App Runner

### What it is

A managed service to run a web service from source or a container with minimal setup.

Use App Runner when:

- You want a simple “deploy a web service” experience
- You don’t want to manage ECS/EKS details

---

## 8. Lightsail

### What it is

Simplified VPS + networking + storage bundles.

Use Lightsail when:

- You want the simplest compute for small projects
- You don’t need fine-grained VPC architecture or advanced integrations

---

## 9. AWS Batch

### What it is

Managed batch processing built around job queues and scalable compute.

Key concepts

- **Job definition**: container + resources
- **Job queue**: priority + ordering
- **Compute environment**: managed fleet of instances (can include Spot)

Use Batch when:

- Large-scale batch jobs, retries, queueing, and fleet management matter

---

## 10. Hybrid / Edge / Specialized Compute Locations

### Outposts

- AWS-managed infrastructure in your data center
- Use when you need **low latency to on-prem** or data residency constraints

### Local Zones

- AWS compute closer to a specific metro area for lower latency

### Wavelength

- AWS compute at the 5G edge (telecom networks)

---

## 11. Common Compute Comparisons (Exam-Friendly)

### EC2 vs ECS/Fargate vs Lambda

- **EC2**: most control, most ops
- **ECS/Fargate**: containerized workloads, less ops, good for microservices
- **Lambda**: event-driven, minimal ops, great scaling, execution time limits

### ALB vs NLB

- **ALB**: HTTP/HTTPS, routing rules, WAF integration, microservices routing
- **NLB**: TCP/UDP/TLS, static IP, extreme performance, preserves client IP

### ECS vs EKS

- **ECS**: simpler, AWS-native orchestration
- **EKS**: Kubernetes compatibility, broader ecosystem, more complexity

---

## 12. Security Checklist for Compute

- Prefer **IAM roles** over access keys.
- Use **least privilege** for instance/task/function roles.
- Store secrets in **Secrets Manager** (or SSM Parameter Store where appropriate).
- Use **private subnets** for app/data tiers; expose only the load balancer in public subnets.
- Add **VPC endpoints** for AWS services to reduce NAT cost and keep traffic private (where applicable).

---

## 13. Reference Architectures (Compute-Centric)

### Classic web tier (EC2)

```
[Users] -> [ALB] -> [ASG EC2 instances] -> [RDS/Aurora]
```

### Container web tier (ECS Fargate)

```
[Users] -> [ALB] -> [ECS Service (Fargate tasks)] -> [RDS/DynamoDB]
```

### Serverless web/API

```
[Users] -> [API Gateway / ALB] -> [Lambda] -> [DynamoDB/RDS]
```

---

## 14. SAA-C03 “Tell” Phrases to Watch For

- “**Static IP required**” → NLB (or EIP on an instance, but NLB is the common scalable answer)
- “**Path-based routing** / multiple microservices behind one endpoint” → ALB
- “**Spiky / unpredictable traffic**” → Lambda (or Fargate)
- “**Run containers, no server management**” → ECS on Fargate
- “**Need Kubernetes**” → EKS
- “**Batch jobs / queues / large fleet**” → AWS Batch
- “**Need OS-level control**” → EC2
