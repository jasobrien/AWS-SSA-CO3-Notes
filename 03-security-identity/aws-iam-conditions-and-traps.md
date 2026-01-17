# IAM Policy Evaluation + Conditions (SAA-C03) — High-Yield Traps

This note complements the policy-types overview by focusing on **how policies actually get decided** and the **condition patterns** that show up in SAA-C03 questions.

---

## 1. Evaluation basics (what wins?)

1. **Explicit deny** beats everything.
2. Otherwise, you need an **explicit allow** (default is deny).
3. Permissions may be limited by guardrails:
   - **SCPs** (Organizations)
   - **Permissions boundaries**
   - **Session policies** (STS)

Exam “tell”:

- “**Even though the IAM policy allows it, it’s still denied**” → check SCP / boundary / explicit deny.

---

## 2. Identity policy vs resource policy (common exam framing)

- **Identity-based policy**: attached to user/group/role.
- **Resource-based policy**: attached to the resource (S3 bucket policy, KMS key policy, SQS queue policy, SNS topic policy, etc.).

High-yield:

- Cross-account access is often easiest with a **resource policy** (when supported).

---

## 3. High-yield condition patterns

### 3.1 Require MFA

Use when a question says:

- “Require MFA to delete/modify critical resources.”

Concept:

- Condition: `aws:MultiFactorAuthPresent`.

### 3.2 Restrict by source IP

Use when:

- “Only allow from corporate public IP range.”

Concept:

- Condition: `aws:SourceIp`.

### 3.3 Restrict by VPC endpoint (private access)

Use when:

- “Allow S3 access only from private subnets using a VPC endpoint.”

Concept:

- Condition: `aws:sourceVpce` (commonly used with S3 bucket policies and endpoint policies).

### 3.4 ABAC (tag-based access control)

Use when:

- “Teams should only access resources tagged with their team/project.”

Concept:

- Condition keys involving `aws:PrincipalTag/*` and resource tags.

### 3.5 Confused deputy protection (ExternalId)

Use when:

- “Third-party SaaS vendor assumes a role in your account.”

Concept:

- Require **ExternalId** in the trust policy to avoid the confused deputy problem.

Exam “tell”:

- “**Third party needs access via AssumeRole**” → trust policy + ExternalId.

---

## 4. KMS note (why it’s special)

KMS is a frequent trap because access is controlled by:

- **KMS key policy** (primary)
- plus IAM permissions (depending on how the key policy is written)

Exam “tell”:

- “**Cross-account decrypt/encrypt with KMS**” → key policy must allow the external principal.

---

## 5. Fast checklist for exam questions

- Is there any **explicit deny**?
- Is there an **SCP / permissions boundary / session policy** limiting it?
- Is the access **cross-account** (resource policy or trust policy likely needed)?
- Is the question really about **conditions** (MFA, source IP, VPC endpoint, tags, ExternalId)?
