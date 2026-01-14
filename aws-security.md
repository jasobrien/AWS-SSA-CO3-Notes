# AWS Security (SAA-C03) — Detailed Notes

This document covers the AWS security services and design patterns that commonly appear in SAA-C03 scenario questions: identity & access management, encryption, network security, monitoring/detection, and governance.

---

## Quick Decision Guide

- **Need to control who can do what in AWS?** → **IAM** (users/roles/policies) + **Organizations/SCPs** for guardrails
- **Need short-lived credentials for services or users?** → **IAM Roles + STS**
- **Need SSO for workforce identities?** → **IAM Identity Center (AWS SSO)**
- **Need encryption keys you control and audit?** → **KMS**
- **Need secrets rotation for DB credentials/API keys?** → **Secrets Manager**
- **Need centralized logging of API activity?** → **CloudTrail**
- **Need continuous config/compliance evaluation?** → **AWS Config**
- **Need threat detection from logs (CloudTrail/VPC/DNS)?** → **GuardDuty**
- **Need consolidated findings across accounts/services?** → **Security Hub**
- **Need to protect web apps from L7 attacks (SQLi/XSS) & rate limiting?** → **WAF** (often with CloudFront/ALB)
- **Need DDoS protection?** → **Shield** (Standard by default; Advanced for higher protection/support)

---

## 1. Shared Responsibility Model (Know the Boundary)

AWS is responsible for **security OF the cloud** (facilities, hardware, managed service infrastructure). You are responsible for **security IN the cloud** (data, identities, configurations).

Common exam implication:

- Misconfigured IAM policies, public S3 buckets, exposed security groups, missing encryption → **customer responsibility**.

---

## 2. IAM (Identity and Access Management)

### Core concepts

- **Principals**: users, roles, federated identities, AWS services
- **Policies**: JSON documents that grant/deny actions
- **Authentication vs Authorization**:
  - Authentication: who are you?
  - Authorization: what are you allowed to do?

### Users, groups, and roles

- **IAM Users**: long-lived identities (human or legacy app use)
- **IAM Groups**: permissions management for users
- **IAM Roles**: assumed identities with **temporary credentials** (preferred for apps/services)

Exam best practice:

- Use **roles** for EC2/ECS/Lambda and cross-account access.
- Avoid storing access keys on instances or in code.

### Policy evaluation (high-yield)

Policies are evaluated with:

- **Explicit deny** wins
- Otherwise **explicit allow** grants access
- Default is **implicit deny**

### Policy types

- **Identity-based policies**: attached to users/groups/roles
- **Resource-based policies**: attached to resources (e.g., S3 bucket policies, KMS key policies)
- **Permission boundaries**: maximum permissions a role/user can have
- **SCPs (Organizations)**: account-level guardrails (see section 8)

### STS (Security Token Service)

- Provides **temporary credentials** when assuming roles.

Exam “tell”:

- “**Grant access to a user in another AWS account**” → Assume role + trust policy (or resource policy depending on service)

### Common IAM patterns

#### Service-to-service access

- EC2 → IAM role (instance profile)
- ECS task → task role
- Lambda → execution role

#### Cross-account access

- Role in target account trusts source account principal
- Source assumes role via STS

#### MFA

- Strongly recommended for privileged users
- Can be required for sensitive actions via IAM conditions

---

## 3. AWS IAM Identity Center (SSO)

### What it is

Centralized workforce identity management (users/groups/permission sets) for AWS accounts and apps.

Use when:

- You want **SSO** across multiple AWS accounts
- You want to integrate with an external IdP (SAML/OIDC) or manage identities centrally

Exam “tell”:

- “**Company needs single sign-on to multiple AWS accounts**” → IAM Identity Center + Organizations

---

## 4. Encryption and Key Management

### KMS (Key Management Service)

Managed service for creating and controlling encryption keys.

Key points:

- Many AWS services integrate with KMS for encryption at rest
- Central auditing via CloudTrail
- Access control via **key policy** + IAM policies

Exam “tell”:

- “**Need customer-managed keys with auditing and rotation**” → KMS CMK (customer managed key)

### SSE-S3 vs SSE-KMS (common pattern)

- **SSE-S3**: simplest S3-managed keys
- **SSE-KMS**: KMS keys, additional access control and audit trail

### CloudHSM

Dedicated hardware security module.

Use when:

- Strict compliance requires HSM control and you must manage keys in dedicated hardware.

### Encryption in transit

- Use TLS everywhere you can (ALB/CloudFront/API Gateway)
- Use ACM (AWS Certificate Manager) for managed certificates

---

## 5. Secrets and Credential Storage

### Secrets Manager

- Purpose-built for secrets (passwords/API keys)
- Supports **rotation** (e.g., RDS)

Use when:

- “**Rotate DB credentials automatically**” → Secrets Manager

### Systems Manager Parameter Store

- Configuration storage; can store secrets (SecureString) with KMS

Rule of thumb for exams:

- If rotation is explicitly required → **Secrets Manager**
- If you just need encrypted parameters/config → **Parameter Store**

---

## 6. Network Security Controls (Security Groups, NACLs, WAF)

### Security Groups (SG)

- Stateful allow-list firewall at ENI/resource level
- Best for controlling east-west and north-south traffic inside VPC

### NACLs

- Stateless subnet-level controls with allow/deny
- Use when you need explicit denies or subnet-wide policies

### WAF (Web Application Firewall)

- Layer 7 protections for web apps (SQL injection, XSS, bot control, rate limiting)
- Typically attached to **CloudFront** or **ALB**

Exam “tell”:

- “**Block malicious HTTP requests (SQLi/XSS)**” → WAF
- “**Block a specific IP/CIDR at subnet edge**” → NACL deny

### Shield

- **Shield Standard**: automatic DDoS protection for AWS-managed endpoints
- **Shield Advanced**: enhanced protection, cost protection, and access to DRT support

Use when:

- “**Need advanced DDoS protection**” → Shield Advanced

---

## 7. Logging, Monitoring, and Detection

### CloudTrail

Records AWS API calls and events.

Key points:

- Essential for auditing and incident response
- Can deliver logs to S3/CloudWatch Logs

Exam “tell”:

- “**Who deleted this resource?**” → CloudTrail

### AWS Config

Records configuration changes and evaluates resources against rules.

Use when:

- “**Continuously enforce and report compliance**” → Config + rules + conformance packs

### GuardDuty

Threat detection service using CloudTrail, VPC Flow Logs, and DNS logs.

Use when:

- “**Detect compromised credentials / unusual API calls / crypto-mining**” → GuardDuty

### Security Hub

Aggregates findings from GuardDuty, Inspector, Macie, and partner tools.

Use when:

- “**Centralize security findings across accounts**” → Security Hub

### Amazon Inspector

Vulnerability management (especially for EC2 and container images).

Use when:

- “**Scan EC2 for vulnerabilities/exposures**” → Inspector

### Macie

Finds sensitive data (PII) in S3 using ML.

Use when:

- “**Identify PII in S3 buckets**” → Macie

---

## 8. Multi-Account Governance (Organizations, SCPs)

### AWS Organizations

Used to manage multiple accounts with consolidated billing and policy controls.

### Service Control Policies (SCPs)

Guardrails that define the **maximum permissions** for accounts/OUs.

Important:

- SCPs don’t grant permissions; they only limit what IAM can allow.

Exam “tell”:

- “**Prevent all accounts from disabling CloudTrail**” → SCP deny on `cloudtrail:StopLogging` / `cloudtrail:DeleteTrail`
- “**Ensure only approved regions are used**” → SCP with region condition keys

---

## 9. Common Security Patterns

### Least privilege

- Start with minimal permissions and expand as needed.
- Prefer resource-scoped permissions over `Resource: "*"`.

### Separation of duties

- Use multiple accounts and roles to separate environments (dev/test/prod) and responsibilities.

### Break-glass access

- Keep an emergency admin role with MFA and strict monitoring.

### Centralized logging account

- Send CloudTrail/Config logs to a dedicated security/log-archive account.

---

## 10. SAA-C03 “Tell” Phrases to Watch For

- “**Temporary credentials**” → STS / IAM roles
- “**Cross-account access**” → AssumeRole trust policy
- “**Audit API calls**” → CloudTrail
- “**Detect suspicious activity**” → GuardDuty
- “**Compliance / config drift**” → AWS Config
- “**Rotate secrets automatically**” → Secrets Manager
- “**Encrypt with customer-managed keys**” → KMS
- “**Protect web app from SQL injection**” → WAF
- “**DDoS protection needed**” → Shield Advanced
- “**Prevent actions across accounts**” → SCPs
