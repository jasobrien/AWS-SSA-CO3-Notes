# AWS Content Delivery & Edge (SAA-C03) — Detailed Notes

This document focuses on CloudFront and related edge services that improve performance and security for web applications.

---

## Quick Decision Guide

- **Need CDN caching for static/dynamic content?** → **CloudFront**
- **Need protect web app at L7 (SQLi/XSS, rate limiting)?** → **WAF** (often with CloudFront)
- **Need DDoS protection** → **Shield** (Standard by default; Advanced if explicitly required)
- **Need global static IP + faster TCP/UDP failover** → **Global Accelerator**
- **Need DNS-based global routing** → **Route 53**

---

## 1. Amazon CloudFront

### What it is

A global content delivery network (CDN) that caches content at edge locations.

Common origins:

- S3 (static assets)
- ALB / EC2 / ECS service
- API Gateway

### Why CloudFront is a common exam answer

- Lower latency to end users
- Reduced load on origins
- Can add security controls at the edge (WAF, TLS, signed URLs)

### Caching basics

- CloudFront caches responses based on:
  - path
  - headers/cookies/query strings (if configured)
  - TTL settings

Exam “tell”:

- “**Global users, slow website**” → CloudFront

### Behaviors

- Different cache behaviors per path pattern (e.g., `/static/*` vs `/api/*`).

### Security and access control

- Use HTTPS via CloudFront certificates.
- For S3 origins, prefer modern origin access controls to keep buckets private.
- Use **signed URLs/cookies** when only authorized users should access private content.

Exam “tell”:

- “**Private content only for logged-in users**” → signed URLs/cookies

---

## 2. AWS WAF (Edge/Web Protection)

### What it is

Web Application Firewall for L7 protections.

Use when:

- SQL injection and XSS protection
- Rate limiting
- IP reputation filtering

Common attachment points:

- CloudFront
- ALB

Exam “tell”:

- “**Protect from SQL injection**” → WAF

---

## 3. AWS Shield (DDoS)

- **Shield Standard**: included, baseline DDoS protection
- **Shield Advanced**: enhanced protection and support

Exam “tell”:

- “**Need DDoS protection with advanced features/support**” → Shield Advanced

---

## 4. Route 53 vs Global Accelerator (Short Comparison)

### Route 53

- DNS-based routing policies (latency, failover, weighted)

### Global Accelerator

- Uses AWS global network with static anycast IPs
- Improves performance and failover for TCP/UDP applications

Exam “tell”:

- “**Static IP + fast global failover**” → Global Accelerator
- “**DNS-based failover/weighted traffic**” → Route 53

---

## 5. Common Edge Patterns (Exam-Friendly)

### Static site

```
[Users] -> [CloudFront] -> [S3]
```

### Web app with dynamic API

```
[Users] -> [CloudFront] -> [ALB/API Gateway] -> [App]
                     \-> [S3 static assets]
```

---

## 6. SAA-C03 “Tell” Phrases to Watch For

- “**Reduce latency globally**” → CloudFront
- “**Reduce origin load**” → CloudFront caching
- “**Protect against SQL injection/XSS**” → WAF
- “**DDoS protection required**” → Shield Advanced
- “**Static IP for global app endpoints**” → Global Accelerator
- “**Fail over between regions**” → Route 53 failover routing
