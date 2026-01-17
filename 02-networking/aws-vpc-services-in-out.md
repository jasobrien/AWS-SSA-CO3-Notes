# AWS Services: Inside vs Outside a VPC (SAA-C03) — Detailed Notes

This note answers a common exam/architecture question:

- **What services/resources are “in” your VPC (have private IPs/ENIs in your subnets)?**
- **What services live outside your VPC (AWS-managed endpoints), even when you access them privately?**

---

## Quick Decision Guide

- **Need to know if something is “in the VPC”?** → Ask: **Does it get a private IP in my subnet / create ENIs in my VPC?**
- **Need private access from a VPC to a service that’s “outside”?** → Use **VPC Endpoints**
  - **S3 + DynamoDB** → **Gateway endpoint**
  - **Most others (STS, SSM, KMS, CloudWatch, SQS, SNS, Secrets Manager, ECR, …)** → **Interface endpoint (PrivateLink)**
- **Need AWS-managed compute to access VPC resources?** (Lambda/Fargate) → Put it **in a VPC** (it creates ENIs), or connect via a managed integration (e.g., API Gateway → VPC Link)

---

## 1. What “inside a VPC” actually means

A VPC is your **networking boundary**. A resource is “in your VPC” when it:

- Lives in **your subnets**
- Gets **private IPs** from your VPC CIDR
- Uses **Elastic Network Interfaces (ENIs)** attached into your VPC

A lot of AWS services are **regional managed services** whose **control plane** is outside your VPC, even if the **data plane** attaches into your VPC.

High-yield phrasing:

- “**In the VPC**” usually means _data plane has ENIs in subnets_.
- “**Outside the VPC**” usually means _you reach it via AWS public/regional endpoints_, optionally via **PrivateLink**.

---

## 2. Services/resources that are (data-plane) “in your VPC”

These commonly deploy into subnets and participate directly in your VPC routing/Security Groups/NACLs.

| Category             | Examples (typical “in VPC” resources)                                                                     | Notes                                                                                    |
| -------------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Compute              | **EC2 instances**, **EC2 Auto Scaling**, **ECS on EC2**                                                   | Classic “in VPC” resources with ENIs/private IPs.                                        |
| Containers           | **ECS tasks (awsvpc mode)**, **AWS Fargate tasks**                                                        | Tasks get ENIs in your subnets.                                                          |
| Kubernetes           | **EKS worker nodes** (and pods via CNI)                                                                   | **EKS control plane** is AWS-managed outside your VPC, but workers/pods are in your VPC. |
| Load balancing       | **ALB**, **NLB**, **Gateway Load Balancer (GWLB)**                                                        | Load balancers get ENIs in selected subnets.                                             |
| Databases            | **RDS DB instance / cluster**, **Aurora**, **RDS Proxy**                                                  | Deployed in DB subnets; “publicly accessible” is a setting, not a different service.     |
| Caches               | **ElastiCache (Redis/Memcached)**                                                                         | Nodes live in subnets.                                                                   |
| Analytics            | **Redshift** (when deployed in a VPC)                                                                     | Redshift is VPC-deployable; clients usually in VPC too.                                  |
| Search               | **OpenSearch Service** (VPC access)                                                                       | Deployed into subnets with security groups.                                              |
| Streaming            | **MSK (Managed Kafka)**                                                                                   | Brokers in your subnets.                                                                 |
| File storage         | **EFS Mount Targets**, **FSx file systems**                                                               | The mount targets/endpoints exist in your subnets.                                       |
| Directory/VDI        | **AWS Directory Service** (VPC), **WorkSpaces**                                                           | Live in subnets; integrate with AD networking.                                           |
| Private connectivity | **Transit Gateway attachments**, **VPC Peering**, **Site-to-Site VPN**, **Direct Connect Gateway (edge)** | The network attachments integrate with your route tables.                                |

Exam “tell”:

- “**Place X in private subnets**” → the thing must be VPC-deployable (ENIs in subnets).
- “**Use security groups to restrict access**” → implies the resource uses ENIs and security groups (i.e., VPC data plane).

---

## 3. Services that live “outside the VPC” (but you can access them privately)

These services do **not** run inside your VPC. You typically reach them via:

- Public AWS endpoints over the internet (often via **IGW/NAT**)
- Or **VPC Endpoints** to keep traffic private

### 3.1 Gateway endpoints (special case)

| Service      | Endpoint type        | Notes                                                            |
| ------------ | -------------------- | ---------------------------------------------------------------- |
| **S3**       | **Gateway endpoint** | Adds routes to route tables; great for “no NAT” private subnets. |
| **DynamoDB** | **Gateway endpoint** | Same idea as S3.                                                 |

### 3.2 Interface endpoints (PrivateLink)

Common “outside” services frequently accessed from private subnets via interface endpoints:

- **STS** (AssumeRole / credential vending)
- **SSM** (Session Manager), **EC2 Messages**, **SSM Messages**
- **CloudWatch Logs**, **CloudWatch** (varies by feature)
- **KMS**
- **Secrets Manager**, **Parameter Store**
- **ECR (API + DKR)**
- **SNS**, **SQS**
- **API Gateway** (private API endpoints)

Exam “tell”:

- “**Instances in private subnets must access AWS APIs without internet/NAT**” → you need **interface endpoints** (and often S3 gateway endpoint too).

---

## 4. Services that are “outside” by default, but can be attached to your VPC

These are AWS-managed services where the service itself is regional/managed, but it can create ENIs in your subnets to reach private resources.

| Service         | VPC relationship                                       | Key gotchas                                                                               |
| --------------- | ------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| **Lambda**      | Can run **in a VPC** (creates ENIs)                    | Without VPC endpoints/NAT, a VPC-attached Lambda can lose access to public AWS endpoints. |
| **API Gateway** | Outside VPC; can reach VPC via **VPC Link** to NLB     | API Gateway itself isn’t “in VPC.”                                                        |
| **App Runner**  | Outside by default; can connect to VPC (VPC connector) | Think “managed ingress,” with optional VPC egress.                                        |
| **Glue**        | Outside by default; can use VPC for data sources       | Similar pattern: managed service + optional VPC connectivity.                             |
| **EKS**         | Control plane outside; workers in VPC                  | Exam trap: “EKS cluster endpoint” is not inside your subnets.                             |

---

## 5. Services that are essentially “outside the VPC” (global/regional control plane)

These are usually not discussed as “in VPC” services, even though they interact with VPC resources.

- **IAM** (global)
- **Route 53** (global service; resolver endpoints can be in VPC)
- **CloudFront** (edge/global)
- **WAF / Shield** (edge/global components)
- **CloudTrail** (regional trails + global service events)
- **Organizations / Control Tower**

---

## 6. Practical exam patterns (high-yield)

### Pattern A: “Private subnets, no internet egress”

To let workloads still function, you typically need endpoints for:

- **S3 (gateway endpoint)**
- **STS** (for role credentials)
- **SSM + EC2 Messages + SSM Messages** (for Session Manager)
- **ECR API + ECR DKR** (if pulling images)
- **CloudWatch Logs** (if shipping logs)
- **KMS / Secrets Manager** (if using managed encryption/secrets)

### Pattern B: “Is this service inside my VPC?”

If it has:

- **subnet selection** + **security groups** + **private IPs**

…it’s effectively “in your VPC” from a networking perspective.

---

## 7. One-sentence rule of thumb

If the thing **gets an ENI in your subnet**, it’s “in your VPC”; otherwise it’s “outside,” and you access it via **internet/NAT** or **VPC endpoints/PrivateLink**.
