# AWS Gateways & Endpoints (SAA-C03) — Detailed Notes

This chapter is a quick-reference to avoid common SAA-C03 confusion between:

- **Networking gateways** (IGW/NAT/egress-only IGW) — VPC edge routing
- **Hybrid connectivity gateways** (VGW/CGW/TGW/DX) — network-to-network connectivity
- **Service-level gateways** (API Gateway, Storage Gateway) — managed services (not route targets)
- **Endpoints** (VPC endpoints / PrivateLink, Route 53 Resolver endpoints) — private service/DNS access

---

## Quick Decision Guide

- **Need inbound + outbound internet for a public subnet?** → **Internet Gateway (IGW)** + public subnet route + public IPv4/EIP
- **Need outbound-only IPv4 internet from private subnets?** → **NAT Gateway** (in a public subnet)
- **Need outbound-only IPv6 internet from private subnets?** → **Egress-only Internet Gateway**
- **Need private access to AWS services without using internet/NAT?** → **VPC Endpoints**
  - **S3/DynamoDB** → **Gateway endpoint**
  - **Most other services** → **Interface endpoint (PrivateLink)**
- **Need on-prem ↔ AWS encrypted connectivity quickly** → **Site-to-Site VPN** (CGW + VGW/TGW)
- **Need many VPCs/accounts + hub routing (transitive)** → **Transit Gateway (TGW)**
- **Need predictable hybrid bandwidth/latency** → **Direct Connect (DX)** (often with VPN backup)
- **Need hybrid DNS resolution (on-prem ↔ VPC)** → **Route 53 Resolver endpoints** (inbound/outbound)

---

## 1. Networking gateways (VPC internet edge)

### Internet Gateway (IGW)

- Attached to a VPC.
- Enables internet routing for subnets whose route tables include `0.0.0.0/0 -> igw-...`.

Checklist for “publicly reachable EC2”:

- Instance in a **public subnet** (route to IGW)
- Instance has a **public IPv4** or **Elastic IP**
- Security group/NACL allow required inbound

Exam “tell”:

- “**Publicly accessible website**” → IGW + public subnet + public IP/EIP

Common confusion:

- IGW does not automatically make a resource public — you still need the route and a public address.

### NAT Gateway

- Provides **outbound-only IPv4** internet access for private subnets.
- Deployed in a **public subnet** and typically uses an **Elastic IP**.

High-yield HA note:

- NAT Gateways are **AZ-scoped** → for AZ resilience use **one NAT GW per AZ** and route each private subnet to its same-AZ NAT.

Exam “tell”:

- “**Private instances need to download patches / reach the internet**” → NAT Gateway

Common confusion:

- NAT Gateway does not accept inbound connections initiated from the internet.

### Egress-only Internet Gateway (IPv6)

- Enables **outbound-only IPv6** internet access.
- Use when you want IPv6 egress but no inbound IPv6 initiated traffic.

Exam “tell”:

- “**IPv6 private subnets need outbound internet but block inbound**” → egress-only IGW

---

## 2. Hybrid connectivity gateways (network-to-network)

### Customer Gateway (CGW)

- Represents your **on-prem VPN device** (your side).

### Virtual Private Gateway (VGW)

- VPN endpoint on the AWS side, attached to a VPC.
- Used for **Site-to-Site VPN** into one VPC (classic model).

Exam “tell”:

- “**Encrypted tunnels over the internet to a VPC**” → Site-to-Site VPN (CGW + VGW)

### Transit Gateway (TGW)

- Hub router for many VPCs and hybrid attachments.
- Supports **transitive routing**.

Exam “tell”:

- “**Many VPCs need connectivity / hub-and-spoke**” → TGW
- “**Transitive routing required**” → TGW (not VPC peering)

### Direct Connect (DX)

- Dedicated private connectivity to AWS with more consistent performance.

High-yield:

- **Not encrypted by default** (VPN can be layered for encryption).
- For availability, use **redundant DX connections** and often a **VPN backup path**.

---

## 3. VPC endpoints (private access to AWS services)

Goal: reach AWS services from a VPC **without traversing the public internet**.

### Gateway VPC endpoints

- Only for **S3** and **DynamoDB**.
- Implemented via **route table entries**.

Exam “tell”:

- “**Private subnets need S3 without NAT**” → S3 gateway endpoint

### Interface VPC endpoints (AWS PrivateLink)

- Create **ENIs in your subnets** with private IPs.
- Used for many services (STS/SSM/KMS/CloudWatch Logs/ECR/SQS/SNS/Secrets Manager, etc.).

Exam “tell”:

- “**Private subnets must call AWS APIs without NAT/IGW**” → interface endpoints

Common confusion:

- Interface endpoints are not VPC peering — they provide private access to specific services, not full network connectivity.

### PrivateLink endpoint services (provider side)

- Lets you expose a service privately (commonly behind an NLB) to other VPCs/accounts.

Exam “tell”:

- “**Expose an internal service privately to another account without peering**” → PrivateLink endpoint service

---

## 4. DNS endpoints (Route 53 Resolver endpoints)

These are about **DNS query forwarding** between on-prem and VPCs.

### Inbound Resolver endpoint

- Allows **on-prem** to resolve names in **private hosted zones** in a VPC.

### Outbound Resolver endpoint

- Allows **VPC** resources to resolve **on-prem domains** via forwarding rules.

Exam “tell”:

- “**On-prem needs to resolve VPC private DNS**” → inbound endpoint
- “**VPC needs to resolve on-prem DNS**” → outbound endpoint + rules

---

## 5. Service-level “gateways” (not route-table gateways)

These are managed services named “gateway” but they are not VPC edge routing components.

### Amazon API Gateway

- Managed API front door for HTTP(S) APIs (auth, throttling, caching, etc.).
- Can integrate with private backends via **VPC Link** (typically to an NLB).

### AWS Storage Gateway

- Hybrid storage integration:
  - **File Gateway** (NFS/SMB → S3)
  - **Volume Gateway** (iSCSI block)
  - **Tape Gateway** (virtual tapes)

---

## 6. Common exam traps (fast)

- **IGW vs NAT GW**: IGW enables public subnet internet routing; NAT GW provides outbound-only for private subnets.
- **Gateway endpoint vs interface endpoint**: gateway endpoint is S3/DynamoDB + route table; interface endpoint is ENIs + security groups.
- **VPC endpoint vs Resolver endpoint**: VPC endpoints are for service connectivity; resolver endpoints are for DNS.
- **TGW vs VPC peering**: peering is non-transitive; TGW is hub-and-spoke with transitive routing.
