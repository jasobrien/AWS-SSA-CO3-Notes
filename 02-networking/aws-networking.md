# AWS Networking (SAA-C03) — Detailed Notes

This document focuses on AWS networking: VPC fundamentals, routing, security boundaries, private connectivity, and DNS patterns that commonly appear in SAA-C03 scenario questions.

---

## Quick Decision Guide

- **Need an isolated network for AWS resources?** → **VPC**
- **Need internet access for resources in a public subnet?** → **Internet Gateway (IGW) + public route**
- **Need outbound-only internet for private subnets?** → **NAT Gateway** (or NAT instance, rarely)
- **Need private access to AWS services without using the internet/NAT?** → **VPC Endpoints**
  - S3/DynamoDB → **Gateway Endpoint**
  - Most other services → **Interface Endpoint (PrivateLink)**
  - (Quick refresher: [Gateways & Endpoints](aws-gateways-and-endpoints.md))
- **Need to connect two VPCs (small number, simple)?** → **VPC Peering**
- **Need hub-and-spoke networking across many VPCs/accounts?** → **Transit Gateway (TGW)**
- **Need connect on-prem to AWS over the internet (encrypted)?** → **Site-to-Site VPN**
- **Need dedicated private connectivity to AWS (consistent bandwidth/latency)?** → **Direct Connect**
- **Need globally available DNS and traffic steering?** → **Route 53**

---

## 1. VPC Fundamentals

### What a VPC is

A logically isolated virtual network in AWS where you control:

- IP addressing (CIDR blocks)
- Subnets, routing, and gateways
- Network security controls

### IPv4 CIDR planning (high-yield)

- VPC has an IPv4 CIDR (e.g., `10.0.0.0/16`).
- Subnets split that CIDR into smaller ranges.
- Avoid overlapping CIDRs if you expect future peering/TGW/on-prem connectivity.

### IPv6 overview

- IPv6 in VPC is typically a /56 assigned by AWS.
- Subnets usually get /64.
- IPv6 is internet-routable by design; use security groups/NACLs carefully.

---

## 2. Subnets: Public vs Private

### Public subnet

A subnet is “public” if its route table contains a route to an **Internet Gateway**:

- `0.0.0.0/0 -> igw-...`

Commonly hosts:

- Load balancers (ALB/NLB)
- Bastion hosts (less common now; SSM is preferred)

### Private subnet

A subnet is “private” if it does not have a direct route to an IGW.

Commonly hosts:

- Application servers (EC2)
- Containers (ECS/EKS)
- Databases (RDS)

Outbound internet from private subnet is typically via:

- `0.0.0.0/0 -> nat-...` (NAT Gateway in a public subnet)

Exam “tell”:

- “**Database should not be internet accessible**” → private subnets + no IGW route

---

## 3. Route Tables and Routing

### How routing works

- Each subnet is associated with exactly one route table.
- Route tables decide where traffic goes.
- The **most specific route** (longest prefix match) wins.

### Common targets

- **Local**: within the VPC CIDR
- **IGW**: internet access for public subnets
- **NAT Gateway**: outbound-only internet for private subnets
- **VPC Peering**: routes to another VPC’s CIDR
- **Transit Gateway**: hub routing to many networks
- **VPC Endpoint**: private routes to AWS services

---

## 4. Internet Connectivity

### Internet Gateway (IGW)

- Attached to a VPC.
- Enables resources with **public IPv4** (or EIP) to be reachable from the internet (subject to routes + security rules).

Checklist for “publicly reachable EC2”:

- Instance in **public subnet** (route to IGW)
- Instance has **public IPv4** or **Elastic IP**
- Security group allows inbound as needed
- NACL allows inbound/outbound as needed

### NAT Gateway

- Provides **outbound** internet for private subnets.
- Deployed in a **public subnet**, with a route from private subnets pointing to it.

Key properties:

- Managed service, scales automatically.
- AZ-scoped (for HA, deploy one per AZ and route private subnets to their AZ-local NAT).

Exam “tell”:

- “**Instances in private subnet need to download patches**” → NAT Gateway

---

## 5. Security at the Network Layer: Security Groups vs NACLs

### Security Groups (SG)

- Attached to ENIs (instances, ALBs, ECS tasks in `awsvpc`, etc.)
- **Stateful**: return traffic is automatically allowed
- Rules are **allow-only**

Typical use:

- Allow inbound HTTP/HTTPS to an ALB
- Allow inbound app traffic only from the ALB security group
- Allow DB port only from the app security group

### Network ACLs (NACL)

- Attached to subnets
- **Stateless**: you must allow inbound AND outbound explicitly
- Supports **allow and deny**

Typical use:

- Coarse subnet-level controls, compliance requirements, explicit denies

Exam “tell”:

- “**Block a specific IP at subnet boundary**” → NACL (deny)
- “**Allow app tier to talk to DB tier**” → SG rules referencing SGs

---

## 6. VPC Endpoints (Private Access to AWS Services)

Use VPC endpoints to access AWS services **without traversing the public internet** (and often without NAT).

### Gateway Endpoints

- Only for **S3** and **DynamoDB**.
- Added as routes in your route table.

Benefits:

- Keep traffic private
- Reduce NAT Gateway data processing costs for S3/DynamoDB access from private subnets

### Interface Endpoints (AWS PrivateLink)

- ENIs in your subnets with private IPs.
- Used for many services (e.g., Secrets Manager, SSM, CloudWatch Logs, ECR, KMS, etc.).

Exam “tell”:

- “**Private subnet needs to call Secrets Manager without internet**” → Interface VPC Endpoint
- “**Private subnet needs S3 without NAT**” → Gateway Endpoint

---

## 7. DNS and Traffic Management (Route 53)

### Hosted zones

- **Public hosted zone**: internet-resolvable DNS
- **Private hosted zone**: resolvable only inside associated VPC(s)

### Record types (high-yield)

- **A / AAAA**: IPv4 / IPv6
- **CNAME**: maps a **name → another name** (e.g., `app.example.com` → `d123.cloudfront.net`)
  - Cannot be used at the **zone apex** (root) (e.g., `example.com`) because apex must have SOA/NS records
  - Typically introduces an additional DNS resolution step (client resolves CNAME target next)
  - Works for any DNS name (inside or outside AWS), but not valid when you need an apex record
- **ALIAS** (Route 53 feature): Route 53-only “name → AWS target” record that behaves like **A/AAAA** to the client
  - **Can be used at the zone apex** (e.g., `example.com` → CloudFront / ALB)
  - Lets you point at AWS resources whose IPs change, without hardcoding addresses
  - Common alias targets: **CloudFront**, **ALB/NLB**, **API Gateway**, **S3 website endpoint**, **Global Accelerator**, and other Route 53 records in the same hosted zone
  - Generally preferred over CNAME for AWS resources; Route 53 can answer with the current target IPs directly (no “CNAME hop” exposed to the client)
  - Route 53 ALIAS queries are **not charged** as “Route 53 queries” (helpful exam detail: CNAME/A are normal queries)
  - Not a standard DNS record type outside Route 53 (if you move DNS providers, you’ll re-model this)

Quick exam heuristics:

- “**Root domain needs to point to CloudFront/ALB**” → use **A/AAAA ALIAS** (not CNAME)
- “**Subdomain points to another DNS name**” → often **CNAME** (unless you want ALIAS to an AWS target)

Example (typical web setup):

- `example.com` → **A (ALIAS)** → `d123456abcdef8.cloudfront.net`
- `www.example.com` → **CNAME** → `example.com` (or directly to `d123456abcdef8.cloudfront.net`)

### Routing policies (common exam themes)

- **Simple**: single record
- **Weighted**: split traffic (blue/green)
- **Latency-based**: route to lowest latency region
- **Failover**: active/passive with health checks
- **Geolocation/Geoproximity**: route based on user location
- **Multi-value**: return multiple healthy records

Exam “tell”:

- “**Active-passive failover between regions**” → Route 53 failover + health checks
- “**Split traffic 90/10 to test new version**” → weighted routing

---

## 8. Cross-VPC Connectivity

### VPC Peering

- Point-to-point connection between two VPCs.
- Works across accounts/regions.

Important limitations (frequent exam traps):

- **No transitive routing** (A peers with B, B peers with C does not imply A can reach C).
- **CIDR blocks must not overlap**.

Use when:

- Small number of VPCs, simple connectivity

### Transit Gateway (TGW)

- Hub-and-spoke routing for many VPCs and on-prem connections.
- Supports transitive routing (through the hub).

Use when:

- Many VPCs, many accounts, centralized networking

---

## 9. Connecting On-Prem to AWS

### Site-to-Site VPN

- Encrypted tunnels over the public internet.
- Terminates on a **Virtual Private Gateway (VGW)** or **Transit Gateway**.

Use when:

- You need relatively quick setup and encryption over internet

### Direct Connect (DX)

- Dedicated private network connection to AWS.
- More consistent performance, often higher bandwidth.

Key exam concepts:

- **DX is not encrypted by default** (can layer VPN for encryption).
- Used for hybrid connectivity with predictable throughput/latency.

Exam “tell”:

- “**Need consistent low-latency hybrid connection**” → Direct Connect
- “**Need encryption over DX**” → DX + VPN

### Direct Connect resiliency patterns (high-yield)

Many SAA-C03 questions are really asking for **hybrid connectivity with predictable performance AND high availability**.

Common best-practice answers:

- **Redundant DX connections** (ideally in different locations) for higher availability.
- **Site-to-Site VPN as backup** path (failover) when DX is down.
- Use **Transit Gateway** when many VPCs/accounts need to share the same hybrid connectivity.

Exam “tell”:

- “**Hybrid connectivity must be highly available**” → redundant DX (often + VPN backup)
- “**Need quick failover if dedicated link fails**” → DX + VPN backup

---

## 10. Load Balancers and Networking (Short Reference)

- **ALB**: L7 HTTP/HTTPS, host/path routing, integrates with WAF
- **NLB**: L4 TCP/UDP/TLS, static IPs/EIPs, very high performance

Exam “tell”:

- “**Static IP for the load balancer**” → NLB
- “**Path-based routing**” → ALB

---

## 11. VPC Flow Logs and Observability

### VPC Flow Logs

Capture IP traffic metadata at:

- VPC level
- Subnet level
- ENI level

Common destinations:

- CloudWatch Logs
- S3

Use when:

- Troubleshooting reachability/security group/NACL/routing issues
- Security auditing and anomaly detection

---

## 12. Common Patterns and Exam “Tell” Phrases

- “**Private subnets need S3 access without internet**” → S3 Gateway Endpoint
- “**Private subnets need AWS API calls without NAT**” → Interface Endpoints
- “**Block a specific CIDR**” → NACL deny (or WAF at L7 for HTTP)
- “**Many VPCs need full mesh connectivity**” → Transit Gateway
- “**Two VPCs only need to talk to each other**” → VPC Peering
- “**Hybrid encrypted connectivity quickly**” → Site-to-Site VPN
- “**Dedicated, reliable hybrid connectivity**” → Direct Connect

---

## 13. Reference Architectures (Networking-Centric)

### Standard 2-tier VPC

```
Public Subnets (2+ AZs):   [ALB]  [NAT GW]
Private Subnets (2+ AZs):  [App EC2/ECS] -> [RDS]

Routes:
- Public: 0.0.0.0/0 -> IGW
- Private: 0.0.0.0/0 -> NAT GW
```

### Private service access without NAT

```
[Private Subnet Apps] -> [VPC Endpoints]
  - Gateway: S3/DynamoDB
  - Interface: SSM/Secrets Manager/KMS/CloudWatch Logs/etc.
```
