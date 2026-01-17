# AWS Identity & Governance (SAA-C03) — Detailed Notes

This document focuses on multi-account identity and governance patterns: Organizations, SCPs, cross-account access, and centralized administration.

---

## Quick Decision Guide

- **Need to manage multiple AWS accounts centrally?** → **AWS Organizations**
- **Need guardrails that limit what accounts can do?** → **SCPs**
- **Need workforce SSO to many accounts?** → **IAM Identity Center**
- **Need cross-account access between workloads?** → **AssumeRole (STS)**
- **Need to share resources across accounts (subnets, TGW, etc.)?** → **AWS RAM**

---

## 1. Multi-Account Strategy (Why it matters)

Common reasons to use multiple accounts:

- Isolation (blast radius reduction)
- Separate billing/cost allocation
- Different security controls for prod vs dev

Typical layout:

- Management account (Organizations)
- Shared services account
- Log archive/security account
- Workload accounts (dev/test/prod)

---

## 2. AWS Organizations

### What it is

Central management for multiple AWS accounts.

Key features:

- Consolidated billing
- Organizational Units (OUs)
- Policy application across OUs/accounts (SCPs)

Exam “tell”:

- “**Company has many AWS accounts**” → Organizations

---

## 2.1 AWS Control Tower (Landing Zone)

### What it is

Control Tower is a managed way to set up and govern a multi-account AWS environment (“landing zone”) on top of **Organizations**.

At the exam level, think:

- **Organizations + best-practice baseline** (accounts, guardrails, logging)
- Standard structure that reduces DIY effort for multi-account governance

### Key concepts (exam-level)

- **Guardrails**: pre-packaged governance controls (implemented via SCPs and other mechanisms)
- **Account Factory**: standardized account provisioning
- Common baseline accounts:
	- **Log archive** (centralized logs)
	- **Audit / security** account

### When it’s the best answer

- “Need to **set up and govern** a new multi-account environment quickly”
- “Need **standardized** account provisioning and guardrails”

Exam “tell”:

- “**Landing zone**” / “**set up multiple accounts with guardrails**” → Control Tower

---

## 3. Service Control Policies (SCPs)

### What SCPs do

- Define the **maximum allowed permissions** in an account or OU.

Important:

- SCPs do **not** grant permissions.
- You still need IAM policies to allow actions.
- **Explicit deny** in an SCP cannot be overridden by IAM.

Common guardrails tested:

- Restrict AWS regions
- Prevent disabling CloudTrail/Config
- Deny risky services/actions

Exam “tell”:

- “**Prevent all accounts from doing X**” → SCP

---

## 4. IAM Identity Center (AWS SSO)

### What it is

Centralized workforce access to multiple accounts using permission sets.

Use when:

- You want SSO across accounts
- You want to reduce IAM user sprawl

Exam “tell”:

- “**Employees need SSO to AWS accounts**” → Identity Center

---

## 5. Cross-Account Access (STS AssumeRole)

### Standard pattern

- Target account defines a role with a **trust policy** allowing principals from the source account.
- Source principal assumes the role via STS.

Use when:

- Workloads in one account need to access resources in another.

Exam “tell”:

- “**Access resources in another AWS account securely**” → AssumeRole

---

## 6. Resource Sharing (AWS RAM)

### What it is

Share certain AWS resources across accounts (within Organizations).

Examples:

- Shared subnets (VPC sharing)
- Transit Gateway attachments

Use when:

- Central networking team manages VPC/TGW but workloads live in separate accounts

---

## 7. Common Pitfalls

- Using long-lived IAM users for applications instead of roles
- Assuming SCPs grant permissions
- Not separating log/audit resources into dedicated accounts

---

## 8. SAA-C03 “Tell” Phrases to Watch For

- “**Centralize billing and account management**” → Organizations
- “**Apply global guardrails across accounts**” → SCPs
- “**SSO for employees**” → IAM Identity Center
- “**Workload in Account A needs access to Account B**” → cross-account AssumeRole
- “**Share networking resources across accounts**” → AWS RAM
