# AWS Deployment, IaC, and Automation (SAA-C03) — Detailed Notes

This document covers Infrastructure as Code (IaC) and deployment/automation concepts that show up in SAA-C03: how to provision, deploy safely, and roll back.

---

## Quick Decision Guide

- **Need declarative infrastructure provisioning?** → **CloudFormation**
- **Need higher-level IaC in code (TypeScript/Python/etc.)?** → **CDK** (still produces CloudFormation)
- **Need multi-account stack rollout?** → **CloudFormation StackSets**
- **Need a simple PaaS deployment for web apps?** → **Elastic Beanstalk / App Runner**
- **Need CI/CD pipeline?** → **CodePipeline + CodeBuild + CodeDeploy** (or a third-party CI)

---

## 1. CloudFormation

### What it is

Declarative templates to provision AWS resources as stacks.

### Core features

- **Stacks**: a group of resources managed together
- **Change sets**: preview changes before applying
- **Rollback**: if deployment fails, CFN can roll back
- **Parameters/Outputs**: make templates reusable

Exam “tell”:

- “**Provision consistent infrastructure repeatedly**” → CloudFormation

### Nested stacks

- Break large templates into components.

### StackSets

- Deploy stacks across **multiple accounts and regions**.

Exam “tell”:

- “**Roll out baseline resources to many accounts**” → StackSets

---

## 2. AWS CDK (Cloud Development Kit)

### What it is

IaC using programming languages that synthesizes to CloudFormation.

Use when:

- You want abstraction, reuse, and code-driven infra composition.

---

## 3. Deployment Strategies (High-Yield)

### Rolling updates

- Replace instances/tasks gradually.
- Lower risk than all-at-once.

### Blue/Green

- Two environments (blue=old, green=new).
- Switch traffic, roll back quickly.

Exam “tell”:

- “**Minimize downtime and enable fast rollback**” → blue/green

### Canary

- Send a small percentage of traffic to a new version.

---

## 4. CI/CD Building Blocks

### CodePipeline

- Orchestrates stages (source, build, deploy).

### CodeBuild

- Runs builds/tests.

### CodeDeploy

- Deploys to EC2, Lambda, and ECS (varies by integration).

Exam “tell”:

- “**Automate build/test/deploy**” → CodePipeline + CodeBuild + CodeDeploy

---

## 5. Configuration and Secrets in Deployments

- Use **SSM Parameter Store** or **Secrets Manager** for runtime config.
- Avoid embedding secrets in AMIs, container images, or source control.

---

## 6. Common Pitfalls

- No rollback strategy (all-at-once deployments)
- Manual changes outside IaC leading to drift
- Secrets in code or environment files checked into git

---

## 7. SAA-C03 “Tell” Phrases to Watch For

- “**Repeatable infrastructure**” → CloudFormation/CDK
- “**Preview changes safely**” → Change sets
- “**Deploy to many accounts/regions**” → StackSets
- “**Fast rollback / near-zero downtime deployment**” → Blue/Green
- “**Automated CI/CD**” → CodePipeline/CodeBuild/CodeDeploy
