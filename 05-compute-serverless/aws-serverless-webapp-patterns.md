# Serverless Web App Patterns on AWS (SAA-C03) — Detailed Notes

This document focuses on common serverless web application architectures: static frontends, serverless APIs, authentication, and async processing.

---

## Quick Decision Guide

- **Need a static frontend with global performance?** → **S3 + CloudFront**
- **Need a serverless REST API?** → **API Gateway + Lambda**
- **Need authentication for users?** → **Cognito**
- **Need a serverless database?** → **DynamoDB** (or Aurora Serverless if relational is required)
- **Need async/background processing?** → **SQS + Lambda** (or EventBridge)
- **Need orchestration across steps?** → **Step Functions**

---

## 1. Static Frontend Pattern

### Architecture

```
[Users] -> [CloudFront] -> [S3 (static site assets)]
```

Benefits:

- Highly scalable
- Low operational overhead
- Global caching

Security best practice:

- Keep S3 bucket private and allow access only via CloudFront origin controls.

---

## 2. Serverless API Pattern

### Architecture

```
[Users] -> [API Gateway] -> [Lambda] -> [DynamoDB]
```

When it’s a great fit:

- Spiky or unpredictable traffic
- Many small endpoints
- Event-driven backends

Common concerns:

- Cold starts (mitigate with provisioned concurrency if required)
- Concurrency limits

### API Gateway decision points (high-yield)

Common scenario-driven knobs:

- **Throttling / rate limits**: protect backends from traffic spikes.
- **Caching**: reduce repeated requests and backend load for cacheable endpoints.
- **Auth**: integrate with Cognito (user pools) or use Lambda authorizers when custom logic is required.
- **Private APIs**: expose APIs privately inside a VPC (often combined with interface endpoints / PrivateLink patterns).
- **VPC Link**: when API Gateway must reach **private** backends (typically via **NLB**).

Quick comparison (exam-level):

- **HTTP API**: simpler/lower cost for many common API patterns.
- **REST API**: richer feature set (classic exam answers may still use “REST API” wording).

---

## 3. Authentication and Authorization

### Cognito

Use when:

- You need user sign-up/sign-in and token-based auth for APIs.

Common serverless flow:

- Cognito issues JWT tokens
- API Gateway authorizer validates token
- Lambda executes business logic

Exam “tell”:

- “**Add user login and secure API access**” → Cognito + API Gateway authorizer

---

## 4. Data Layer Choices

### DynamoDB

Best for:

- Key-value/document access patterns
- Very high scale and low ops

### Aurora Serverless (if relational needed)

Best for:

- SQL/relational needs with variable usage patterns

Exam “tell”:

- “**Need SQL joins/ACID**” → Aurora/RDS
- “**Serverless key-value at scale**” → DynamoDB

---

## 5. Async Processing Pattern (Decoupling)

### SQS + Lambda workers

```
[API Lambda] -> [SQS] -> [Worker Lambda] -> [DB]
```

Use when:

- You need to buffer spikes
- You need retries and failure isolation

Include DLQ for poison messages.

---

## 6. Event-Driven Pattern

### EventBridge

Use when:

- Multiple consumers need events
- You need routing rules based on event content

---

## 7. Orchestration Pattern

### Step Functions

Use when:

- Multi-step business processes
- Retries/compensation logic

---

## 8. VPC and Private Resources (Common Exam Trap)

- If Lambda must access resources in private subnets (like RDS), it can be configured for VPC access.
- If that Lambda also needs outbound internet from private subnets, you typically need a NAT Gateway (or VPC endpoints for AWS services).

---

## 9. Common Serverless Reference Architectures

### Full serverless web app

```
[Users] -> [CloudFront] -> [S3 static]
                 \-> [API Gateway] -> [Lambda] -> [DynamoDB]
```

### Serverless + relational

```
[API Gateway] -> [Lambda] -> [RDS/Aurora]
                          \-> [RDS Proxy] (to reduce connection storms)
```

---

## 10. SAA-C03 “Tell” Phrases to Watch For

- “**Serverless REST API**” → API Gateway + Lambda
- “**Static site with global users**” → S3 + CloudFront
- “**Add user authentication**” → Cognito
- “**Handle spikes with buffering**” → SQS + Lambda
- “**Workflow with multiple steps**” → Step Functions
- “**Lambda exhausting DB connections**” → RDS Proxy
