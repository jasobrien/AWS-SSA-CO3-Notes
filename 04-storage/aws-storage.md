# AWS Storage (SAA-C03) — Detailed Notes

This document focuses on AWS storage services and the common “which one do I pick?” patterns that show up in architecture decisions and SAA-C03 questions.

---

## Quick Decision Guide

Start here:

- **Need object storage for files, backups, logs, static sites, data lakes?** → **Amazon S3**
- **Need block storage attached to a single VM?** → **Amazon EBS**
- **Need shared POSIX file system for many instances/containers?** → **Amazon EFS**
- **Need Windows file shares (SMB) or enterprise file system features?** → **Amazon FSx (Windows / ONTAP / OpenZFS)**
- **Need high-performance HPC scratch / parallel file system?** → **FSx for Lustre**
- **Need long-term archival with infrequent access?** → **S3 Glacier storage classes**
- **Need on-prem integration (NFS/SMB/iSCSI) backed by AWS?** → **AWS Storage Gateway**
- **Need to transfer petabytes offline?** → **AWS Snow Family**

---

## 1. Amazon S3 (Simple Storage Service)

### What it is

Durable, highly scalable **object storage**. You store **objects** in **buckets**. Objects have a **key** (name/path-like string) and **metadata**.

S3 is not a file system (no true directories) and not block storage.

### Typical architecture

```
[Apps / Users] -> [S3 Bucket] -> [Lifecycle/Replication/Events]
```

### Storage classes (high-yield)

Pick based on access patterns and retrieval needs:

- **S3 Standard**: frequently accessed
- **S3 Intelligent-Tiering**: unknown/changing access patterns (auto-tiering)
- **S3 Standard-IA**: infrequent access, rapid retrieval
- **S3 One Zone-IA**: infrequent access, single AZ (cheaper, less resilient)
- **S3 Glacier Instant Retrieval**: archive but milliseconds access
- **S3 Glacier Flexible Retrieval**: minutes to hours retrieval
- **S3 Glacier Deep Archive**: lowest cost, hours retrieval (true long-term archive)

Exam “tell”:

- “**Unknown or unpredictable access**” → Intelligent-Tiering
- “**Archive for years, rarely retrieved**” → Glacier Deep Archive
- “**Cheap archive but can wait hours**” → Glacier Flexible Retrieval / Deep Archive

### Durability and availability

- S3 is designed for very high durability (multi-device, multi-AZ replication under the hood).
- Availability depends on storage class.

### Security controls

Layered controls commonly tested:

1. **S3 Block Public Access** (account + bucket)

- Use this to prevent accidental public exposure.

2. **Bucket policy** (resource-based policy)

- Common for granting cross-account access or controlling public access.

3. **IAM policies** (identity-based)

- Grant principals permissions to S3 actions.

4. **Object Ownership / ACLs**

- Modern best practice is to disable ACLs and rely on policies.

5. **Encryption**

- **SSE-S3**: S3-managed keys
- **SSE-KMS**: AWS KMS keys, audit + key policies + finer control
- **Client-side encryption**: you encrypt before upload

Exam “tell”:

- “**Need audit, key rotation, granular key access**” → SSE-KMS
- “**Simplest encryption at rest**” → SSE-S3

### Access patterns & networking

- Use **VPC Gateway Endpoints for S3** to keep traffic off the public internet and avoid NAT data processing costs.
- Use **S3 Access Points** to simplify access control per application/team.

### Private access patterns (CloudFront + VPC endpoints) (high-yield)

These patterns show up in “keep the bucket private” exam scenarios.

#### Pattern A: Private S3 + CloudFront for public content

Goal: users access content via CloudFront, but the S3 bucket is not public.

- Keep **S3 Block Public Access** enabled.
- Use **CloudFront Origin Access Control (OAC)** (modern) or **Origin Access Identity (OAI)** (older) so only CloudFront can read from the bucket.
- Bucket policy allows `s3:GetObject` only for the CloudFront origin identity/control.

Exam “tell”:

- “**Static website must not be publicly accessible in S3**” → CloudFront + OAC/OAI + private bucket

#### Pattern B: Private subnets access S3 without NAT

Goal: workloads in private subnets use S3 without internet egress.

- Use an **S3 Gateway VPC Endpoint**.
- Optionally restrict access with an **endpoint policy** and/or bucket policy conditions.

Common restriction:

- Allow bucket access only via your endpoint using `aws:sourceVpce` (resource policy condition).

Exam “tell”:

- “**Private subnets must access S3 without NAT/IGW**” → S3 gateway endpoint
- “**Ensure S3 access only from your VPC**” → endpoint + bucket policy condition

### Performance & data transfer patterns (high-yield)

These show up frequently in scenario questions about upload speed, global performance, and secure direct uploads.

- **Multipart upload**: best for large objects; improves throughput and allows retrying parts.
- **Pre-signed URLs**: let clients upload/download directly to S3 without proxying through your app servers.
  - Exam framing: “avoid scaling the web tier for file uploads” → pre-signed URL pattern.
- **S3 Transfer Acceleration**: helps **global users upload** to a single region faster.
- **CloudFront in front of S3**: improves **download** performance globally and supports HTTPS/custom domains.

Exam “tell”:

- “**Users worldwide need faster uploads to S3**” → Transfer Acceleration (or region design change)
- “**Avoid routing large uploads through EC2/Lambda**” → pre-signed URLs
- “**Large files and unreliable networks**” → multipart upload

### Versioning and retention

- **Versioning** protects against accidental overwrites/deletes.
- **MFA Delete** can protect version deletions (more administrative friction).
- **S3 Object Lock** (WORM) provides retention controls (governance/compliance mode).

Exam “tell”:

- “**WORM / regulatory retention**” → S3 Object Lock
- “**Protect from accidental deletes/overwrites**” → Versioning

### Lifecycle policies

Automate transitions and expiration:

- Move objects through storage classes as they age.
- Expire old versions and delete markers (with versioning).

### Replication

- **CRR (Cross-Region Replication)**: DR, compliance, latency reduction
- **SRR (Same-Region Replication)**: resilience or log aggregation inside a region

Common requirements:

- Versioning must be enabled for replication.

### Eventing and integrations

- S3 can publish events to **SQS**, **SNS**, **Lambda**, **EventBridge**.

### Static website hosting (common)

- S3 can host static sites, often paired with **CloudFront** for HTTPS + caching + custom domains.

---

## Appendix A: S3 Bucket Policy Snippets (OAC/OAI + `aws:sourceVpce`)

These are intentionally short templates for exam practice. Replace placeholders like `BUCKET`, `ACCOUNT_ID`, `DISTRIBUTION_ID`, and `VPCE_ID`.

### A.1 CloudFront OAC (recommended) — allow CloudFront to read private S3

Use when: “S3 must be private; only CloudFront can access it.”

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontReadViaOAC",
      "Effect": "Allow",
      "Principal": { "Service": "cloudfront.amazonaws.com" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::BUCKET/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::ACCOUNT_ID:distribution/DISTRIBUTION_ID"
        }
      }
    }
  ]
}
```

### A.2 CloudFront OAI (legacy) — allow OAI canonical user to read

Use when: the question explicitly mentions OAI (older wording).

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontReadViaOAI",
      "Effect": "Allow",
      "Principal": { "CanonicalUser": "CLOUDFRONT_OAI_CANONICAL_USER_ID" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::BUCKET/*"
    }
  ]
}
```

### A.3 Allow access only via a specific S3 Gateway VPC Endpoint

Use when: “Only allow S3 access from my VPC / via my endpoint (no internet path).”

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowOnlyFromSpecificVPCE",
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::BUCKET/*",
      "Condition": {
        "StringEquals": {
          "aws:sourceVpce": "vpce-0123456789abcdef0"
        }
      }
    }
  ]
}
```

---

## 2. Amazon EBS (Elastic Block Store)

### What it is

Persistent **block storage** for **EC2**.

Key properties:

- Volumes are **AZ-scoped** (attach within the same AZ).
- Typically attached to **one instance** (some multi-attach support for specific volume types).

### Typical architecture

```
[EC2 Instance] <-> [EBS Volume] -> [Snapshots to S3]
```

### Volume types (high-yield)

- **gp3**: general purpose SSD (default recommendation)
- **io1 / io2**: provisioned IOPS SSD (databases, low-latency + high IOPS)
- **st1**: throughput optimized HDD (large sequential workloads)
- **sc1**: cold HDD (cheapest HDD)

Exam “tell”:

- “**Database needs high IOPS**” → io1/io2
- “**Boot volume / general workloads**” → gp3
- “**Big sequential throughput**” → st1

### Snapshots

- Incremental snapshots stored in S3 (managed by AWS).
- Used for backup, restore, and creating new volumes.
- **Fast Snapshot Restore (FSR)** can reduce restore latency (costs more).

### Encryption

- EBS encryption is easy to enable and commonly expected.
- Snapshots inherit encryption state.

### Limits & gotchas

- One EBS volume cannot be attached across AZs.
- For shared file storage across instances, use **EFS** or **FSx**, not EBS.

---

## 3. Amazon EFS (Elastic File System)

### What it is

Managed **NFS** file system for Linux-like shared storage.

Key properties:

- **Regional** service with mount targets in multiple AZs.
- Many instances can mount the same file system concurrently.

### Typical architecture

```
[EC2/ECS/EKS] -> (NFS) -> [EFS]
```

### Performance and throughput

- EFS scales automatically; performance depends on mode/throughput settings.

### Storage classes

- Standard and IA (infrequent access) via lifecycle management.

### Security

- Security groups control NFS access to mount targets.
- Encryption at rest and in transit are supported.

Exam “tell”:

- “**Shared POSIX file system across many EC2 instances**” → EFS
- “**Web servers need shared content/uploads**” → EFS

---

## 4. Amazon FSx (Managed File Systems)

FSx is used when you need specific file system semantics or enterprise features beyond EFS.

### FSx for Windows File Server

- SMB protocol
- Windows-native features (Active Directory integration)

Use when:

- “**Windows file shares**”, “SMB”, “AD integration”

### FSx for Lustre

- High-performance parallel file system
- Common for HPC, ML training, large-scale analytics
- Often integrated with S3 for data sets

Use when:

- “**HPC**, **very high throughput**, **parallel access**”

### FSx for NetApp ONTAP

- Enterprise NAS features (snapshots, replication, tiering)
- Supports NFS/SMB and storage efficiencies

Use when:

- You need ONTAP capabilities, multi-protocol, enterprise migrations

### FSx for OpenZFS

- ZFS semantics, snapshots, clones

Use when:

- Workloads that benefit from ZFS features and performance characteristics

---

## 5. S3 Glacier (Archival)

Glacier is usually consumed via S3 storage classes:

- **Glacier Instant Retrieval**
- **Glacier Flexible Retrieval**
- **Glacier Deep Archive**

Core trade-offs:

- Lower storage cost
- Higher retrieval latency and potential retrieval fees

Exam “tell”:

- “**Archive + compliance retention**” → Glacier + Object Lock (if WORM needed)

---

## 6. AWS Storage Gateway (Hybrid)

### What it is

Connect on-prem environments to AWS storage using familiar protocols.

### Gateway types (high-yield)

- **File Gateway**: presents NFS/SMB, stores objects in S3
- **Volume Gateway**: iSCSI block storage
  - Cached volumes: cache on-prem, primary in AWS
  - Stored volumes: primary on-prem, async backup to AWS
- **Tape Gateway**: virtual tape library backed by S3/Glacier

Exam “tell”:

- “**Keep using on-prem apps, store in S3**” → File Gateway
- “**Backup software expects tapes**” → Tape Gateway

---

## 7. AWS Snow Family (Offline Transfer + Edge)

Used when networks are too slow/expensive to transfer huge data sets.

- **Snowcone**: small, portable, edge compute/storage
- **Snowball Edge**: larger device for TB–PB scale transfer
- **Snowmobile**: exabyte-scale (rare, extreme cases)

Exam “tell”:

- “**Need to move 100TB+ quickly, limited bandwidth**” → Snowball

---

## 8. Data Transfer and Migration Helpers

### AWS DataSync

- Automated, scheduled data movement between on-prem and AWS (S3/EFS/FSx)
- Handles bandwidth optimization and retries

Use when:

- You need ongoing migration/sync jobs or large-scale transfers over network

### AWS Transfer Family

- Managed SFTP/FTPS/FTP endpoints that land in S3 or EFS

Use when:

- “**Partners need SFTP to upload files into AWS**”

### AWS Backup

- Centralized backup service for multiple AWS services (EBS, EFS, RDS, DynamoDB, etc.)
- Backup vaults, policies, and retention

Use when:

- Standardized backups and governance across accounts/services are required

---

## 9. Common Comparisons (Exam-Friendly)

### S3 vs EBS vs EFS

- **S3**: object storage (HTTP API), unlimited scale, great for files/data lakes
- **EBS**: block device for one EC2 instance (AZ-scoped), good for OS + databases on EC2
- **EFS**: shared file system (NFS) across many compute nodes, multi-AZ

Exam “tell”:

- “**Shared uploads directory across web servers**” → EFS
- “**Persist EC2 root volume / database disk**” → EBS
- “**Store images/videos/logs/backups**” → S3

### EFS vs FSx

- **EFS**: simple managed NFS, elastic
- **FSx Windows**: SMB + AD integration
- **FSx Lustre**: HPC parallel file system
- **FSx ONTAP/OpenZFS**: enterprise features and file system semantics

### Glacier vs Standard-IA

- **Standard-IA**: infrequent but still “normal” retrieval expectations
- **Glacier classes**: archival, cheaper storage, slower retrieval (depending on class)

---

## 10. SAA-C03 “Tell” Phrases to Watch For

- “**Static website hosting**” → S3 + CloudFront
- “**Data lake** / raw objects\*\*” → S3
- “**Shared file system for EC2 fleet**” → EFS
- “**Windows file shares / SMB / Active Directory**” → FSx for Windows
- “**HPC / parallel file system**” → FSx for Lustre
- “**Archive for years**” → Glacier Deep Archive
- “**Hybrid on-prem file shares backed by AWS**” → Storage Gateway (File Gateway)
- “**Move 100TB+ with slow internet**” → Snowball
- “**SFTP uploads into S3**” → Transfer Family
