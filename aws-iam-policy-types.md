# AWS IAM Policy Types and Structure

This document explains the different types of AWS IAM policies, when to use each, and the main sections of a policy document.

---

## 1. Types of IAM Policies

### a. Identity-based Policies

- **Description:** Attach to IAM users, groups, or roles to grant permissions.
- **Use Case:** Grant access to AWS resources for people or applications.
- **Example:** Allow an IAM user to read from an S3 bucket.

### b. Resource-based Policies

- **Description:** Attach directly to AWS resources (e.g., S3 buckets, SQS queues, SNS topics).
- **Use Case:** Allow cross-account access or fine-grained control over a resource.
- **Example:** Allow another AWS account to write to your S3 bucket.

### c. Permissions Boundaries

- **Description:** Advanced feature that sets the maximum permissions an IAM role or user can have.
- **Use Case:** Delegate permission management while limiting the scope of what can be granted.
- **Example:** Allow a developer to create roles, but only with certain permissions.

### d. Service Control Policies (SCPs)

- **Description:** Used with AWS Organizations to set permission guardrails for accounts or organizational units (OUs).
- **Use Case:** Restrict what services/accounts in an organization can do, regardless of their own policies.
- **Example:** Prevent all accounts in an OU from deleting S3 buckets.

### e. Session Policies

- **Description:** Pass policies as part of a session when assuming a role (e.g., with STS AssumeRole).
- **Use Case:** Further restrict permissions for a session, even if the role has broader permissions.
- **Example:** Temporary session with only read access to DynamoDB.

---

## 2. When to Use Each Policy Type

| Policy Type            | Typical Use Case                                     |
| ---------------------- | ---------------------------------------------------- |
| Identity-based         | Grant permissions to users, groups, or roles         |
| Resource-based         | Control access to a specific resource, cross-account |
| Permissions Boundary   | Limit delegated admin permissions                    |
| Service Control Policy | Organization-wide guardrails                         |
| Session Policy         | Temporary, session-specific restrictions             |

---

## 3. Main Sections of an IAM Policy Document

An IAM policy is a JSON document with these main sections:

```json
{
  "Version": "2012-10-17", // Policy language version
  "Statement": [
    {
      "Effect": "Allow" | "Deny", // Whether to allow or deny the action
      "Action": "service:operation" | [ ... ], // Actions this policy applies to
      "Resource": "arn:aws:..." | [ ... ], // Resources this policy applies to
      "Condition": { ... } // (Optional) Conditions for when the policy applies
    }
  ]
}
```

### Section Explanations

- **Version:** Policy language version (always use "2012-10-17" for IAM).
- **Statement:** One or more permission statements.
  - **Effect:** Allow or Deny.
  - **Action:** AWS service actions (e.g., "s3:GetObject").
  - **Resource:** ARN(s) of resources affected.
  - **Condition:** (Optional) Extra requirements (e.g., IP address, MFA).

---

## 4. Example: Simple S3 Read Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

---

**Tip:** Always follow the principle of least privilege—grant only the permissions required for the task.
