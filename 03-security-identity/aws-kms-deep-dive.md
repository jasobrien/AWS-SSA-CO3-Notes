# AWS KMS (Key Management Service) — Deep Dive (SAA-C03)

This chapter covers the KMS concepts that commonly decide SAA-C03 security questions: **key policies vs IAM**, **grants**, **cross-account encryption**, and **service integrations**.

---

## Quick Decision Guide

- **Need audit + tighter control for encryption at rest?** → **SSE-KMS / KMS CMK**
- **Need simplest encryption at rest for S3** → **SSE-S3**
- **Need a third party to use a key in your account** → **KMS key policy + (often) ExternalId on role trust policy**
- **Service needs to use KMS without broad IAM perms** → **KMS grants**

---

## 1. KMS primitives (exam-level)

- **KMS key (CMK)**: logical key in KMS.
- **Key material**: managed by AWS KMS for most exam scenarios.
- **Envelope encryption** (concept): data is encrypted with a data key; the data key is protected by the KMS key.

You typically don’t encrypt large files directly with KMS APIs; services use KMS to protect data keys.

---

## 2. The big trap: Key policy vs IAM policy

KMS authorization is different from many services:

- A principal usually needs:
  - an **IAM permission** allowing `kms:Encrypt`, `kms:Decrypt`, etc.
  - and the **KMS key policy** must allow that principal (directly or indirectly)

High-yield phrasing:

- If a question says “the IAM role has permissions but it still can’t decrypt”, the fix is often **the KMS key policy**.

---

## 3. Grants (why they exist)

A **grant** is a way to allow a principal (often an AWS service) to use a KMS key for a set of operations.

Use cases that show up indirectly in exams:

- Allowing AWS services to use keys without handing out broad `kms:*` permissions.
- Delegating limited key usage.

Exam “tell”:

- “Service can’t use KMS key even with IAM role” → check **key policy** and whether the service relies on **grants**.

---

## 4. Cross-account KMS (high-yield)

Two common patterns:

### Pattern A: Cross-account access to encrypted data (S3, EBS snapshots, etc.)

- Data is in Account A, consumer is in Account B.
- You must allow Account B’s principal to use the key:
  - update **key policy** in Account A
  - and grant the consuming role/user the needed KMS actions via IAM

Exam “tell”:

- “Copy encrypted snapshot / read encrypted S3 objects across accounts” → **key policy changes required**.

### Pattern B: Third-party SaaS assumes a role

- SaaS assumes a role in your account.
- Use **ExternalId** in the role trust policy (confused deputy protection).
- Then allow that role in the **KMS key policy**.

---

## 5. KMS + S3 (SSE-KMS) specifics

- SSE-KMS adds:
  - per-object encryption using a KMS key
  - audit visibility (CloudTrail for KMS usage)
  - key policy control

Common exam interactions:

- **S3 replication** with SSE-KMS often implies **both source and destination keys/policies** need correct permissions.
- Restricting S3 access is often done with:
  - bucket policies (including VPC endpoint conditions)
  - and (separately) KMS policies for decrypt/encrypt.

---

## 6. Common KMS exam “tell” phrases

- “**Need customer-managed keys and auditing**” → KMS
- “**Cross-account decrypt/encrypt fails**” → update KMS key policy
- “**Least privilege for key usage**” → narrow IAM actions + key policy + (sometimes) grants
- “**Compliance requires dedicated HSM**” → CloudHSM (not KMS)

---

## 7. Minimal set of KMS actions (mental model)

- Encrypt path: `kms:Encrypt`, `kms:GenerateDataKey`
- Decrypt path: `kms:Decrypt`
- Often required by services: `kms:DescribeKey`

(Exact sets vary by service; for SAA, this mental model is usually enough.)
