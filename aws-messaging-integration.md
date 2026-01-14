# AWS Messaging & Integration (SAA-C03) — Detailed Notes

This document covers AWS services used to decouple systems and integrate components: queues, pub/sub, event buses, streaming, and orchestration.

---

## Quick Decision Guide

- **Need a queue to decouple producers/consumers?** → **SQS**
- **Need pub/sub fanout to many subscribers?** → **SNS**
- **Need event bus + routing rules for many services?** → **EventBridge**
- **Need workflow orchestration with retries/state?** → **Step Functions**
- **Need managed Kafka-compatible streaming?** → **MSK**
- **Need AWS-native streaming ingestion + shards?** → **Kinesis Data Streams**
- **Need to migrate from on-prem broker (JMS/AMQP) quickly?** → **Amazon MQ**

---

## 1. Amazon SQS (Simple Queue Service)

### What it is

A managed message queue for decoupling components.

Two queue types:

- **Standard**: at-least-once delivery, best effort ordering, highest throughput
- **FIFO**: exactly-once processing (with dedup), strict ordering, lower throughput

Exam “tell”:

- “**Order matters**” → SQS FIFO
- “**Highest throughput queue**” → SQS Standard

### Visibility timeout (high-yield)

When a consumer receives a message, it becomes invisible for the visibility timeout.

- If the consumer fails to delete the message in time, it becomes visible again.

Exam “tell”:

- “**Messages processed twice**” → visibility timeout too short / consumer not deleting

### Dead-letter queues (DLQ)

- Configure a **maxReceiveCount** and send poison messages to a DLQ.

Exam “tell”:

- “**Poison pill / repeatedly failing messages**” → DLQ

### Long polling

- Reduces empty receives and cost.

### Message size and payload patterns

- SQS messages have size limits; large payloads often use the pattern:
  - Put payload in S3, send S3 key in SQS message.

---

## 2. Amazon SNS (Simple Notification Service)

### What it is

A managed pub/sub service.

Common subscribers:

- SQS
- Lambda
- HTTP(S) endpoints
- Email/SMS (less common in architecture exams)

### Fanout pattern (high-yield)

A classic decoupling pattern:

```
[Publisher] -> [SNS Topic] -> [SQS Queue A]
                         -> [SQS Queue B]
                         -> [Lambda]
```

Use when:

- Multiple systems need the same event
- Each consumer should process independently

Exam “tell”:

- “**Send the same message to multiple queues/services**” → SNS fanout

---

## 3. Amazon EventBridge

### What it is

An event bus that routes events based on rules.

Best for:

- Event-driven architectures with many producers/consumers
- Integrations with AWS services and SaaS partners

### Concepts

- **Event bus** (default or custom)
- **Rules**: match event patterns (e.g., source, detail-type)
- **Targets**: Lambda, Step Functions, SQS, SNS, Kinesis, etc.

EventBridge vs SNS (common comparison)

- **SNS**: pub/sub topic, simple fanout
- **EventBridge**: richer routing/filtering, event schema/discovery, multi-bus design

Exam “tell”:

- “**Route events to different targets based on content**” → EventBridge

---

## 4. AWS Step Functions (Orchestration)

### What it is

A state machine service for coordinating distributed workflows.

Good for:

- Multi-step processes with retries and branching
- Coordinating microservices
- Long-running workflows

Exam “tell”:

- “**Orchestrate multiple Lambdas with error handling**” → Step Functions

---

## 5. Streaming

### 5.1 Kinesis Data Streams

- Streaming ingestion with shards.
- Consumers can process records in near real-time.

Use when:

- You need AWS-native streaming and fine control over throughput (shards)

### 5.2 Amazon MSK (Managed Streaming for Apache Kafka)

- Kafka-compatible managed service.

Use when:

- You need Kafka ecosystem compatibility (existing clients, tooling)

---

## 6. Amazon MQ

### What it is

Managed broker for ActiveMQ/RabbitMQ-style integrations.

Use when:

- You need traditional protocols (JMS/AMQP/STOMP/MQTT) for legacy apps
- You want minimal app changes vs rewriting to SNS/SQS/EventBridge

Exam “tell”:

- “**Migrate a legacy message broker with minimal code changes**” → Amazon MQ

---

## 7. Common Patterns (Exam-Friendly)

### Queue-based decoupling

```
[Web/API] -> [SQS] -> [Worker Fleet] -> [DB]
```

### Pub/sub fanout

```
[Producer] -> [SNS] -> [SQS A] -> [Worker A]
                 \-> [SQS B] -> [Worker B]
```

### Event bus routing

```
[Services] -> [EventBridge] --rule--> [Lambda]
                           --rule--> [SQS]
                           --rule--> [Step Functions]
```

---

## 8. SAA-C03 “Tell” Phrases to Watch For

- “**Decouple components with a queue**” → SQS
- “**Fanout to many consumers**” → SNS
- “**Route events by content / event bus**” → EventBridge
- “**Workflow with retries/branching**” → Step Functions
- “**Kafka compatibility**” → MSK
- “**Legacy broker protocols / minimal rewrite**” → Amazon MQ
- “**Poison messages**” → DLQ
