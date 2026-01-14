# AWS Databases (SAA-C03) — Detailed Notes

This document focuses on AWS database choices and the “what should I pick?” decision points commonly tested in SAA-C03.

---

## Quick Decision Guide

- **Need a managed relational database (MySQL/Postgres/etc.)?** → **RDS**
- **Need MySQL/Postgres compatibility with higher performance + advanced HA?** → **Aurora**
- **Need serverless NoSQL key-value/document at massive scale?** → **DynamoDB**
- **Need in-memory caching/session store/sub-millisecond reads?** → **ElastiCache (Redis/Memcached)**
- **Need data warehouse/analytics on structured data?** → **Redshift**
- **Need text search/log analytics with near real-time indexing?** → **OpenSearch Service**
- **Need graph relationships traversal?** → **Neptune**
- **Need MongoDB-compatible managed document DB?** → **DocumentDB**
- **Need time series database (IoT/metrics/events)?** → **Timestream**
- **Need immutable ledger/audit trail with cryptographic verification?** → **QLDB**

---

## 1. Relational Databases

### 1.1 Amazon RDS (Relational Database Service)

**What it is**
Managed relational databases (engine depends on RDS flavor). You manage schema and queries; AWS manages many operational tasks.

Common engines:

- MySQL
- PostgreSQL
- MariaDB
- Oracle
- SQL Server

**High availability (Multi-AZ)**

- **Multi-AZ** provides synchronous replication to a standby in another AZ.
- Automatic failover to standby.

Exam “tell”:

- “**Need HA / automatic failover**” → RDS Multi-AZ

**Read scaling (Read Replicas)**

- **Read replicas** provide asynchronous replicas for scaling reads.
- Used for read-heavy workloads, BI reads, or offloading reporting.

Exam “tell”:

- “**Read-heavy, need horizontal read scaling**” → Read replicas

**Backups**

- Automated backups + point-in-time restore (within retention window)
- Manual snapshots (longer-term / before major changes)

**Security**

- Place RDS in **private subnets**
- Use **security groups** to allow DB port only from app tier
- Encrypt at rest (KMS)
- Prefer credentials in **Secrets Manager**

**RDS Proxy (connection management)**

- Helps with connection pooling, especially for spiky workloads (common with Lambda).

Exam “tell”:

- “**Lambda is exhausting DB connections**” → RDS Proxy

---

### 1.2 Amazon Aurora

**What it is**
A cloud-optimized relational database compatible with **MySQL** and **PostgreSQL**.

**Why it’s often the exam answer**

- Higher performance and resilience (Aurora storage is distributed)
- Supports multiple read replicas and fast failover patterns

**Aurora replicas**

- Typically supports more replicas than standard RDS engines.
- Useful for read scaling.

**Aurora Serverless**

- Auto-scales capacity; useful for intermittent/variable workloads.

Exam “tell”:

- “**Need MySQL/Postgres but better performance/HA than standard RDS**” → Aurora
- “**Intermittent usage, want auto-scaling DB capacity**” → Aurora Serverless

**Global Database**

- Cross-region replication optimized for low-latency global reads and DR.

Exam “tell”:

- “**Global reads + DR across regions**” → Aurora Global Database

---

## 2. NoSQL / Key-Value

### 2.1 Amazon DynamoDB

**What it is**
Fully managed NoSQL database. Commonly used for key-value and document access patterns.

**Core concepts (high-yield)**

- **Table** with a **primary key**:
  - Partition key (required)
  - Sort key (optional)
- Access patterns should be modeled up-front.

**Scaling modes**

- **On-demand capacity**: pay per request; great for spiky/unpredictable traffic
- **Provisioned capacity**: plan capacity; can use auto scaling

**Indexes**

- **GSI** (Global Secondary Index): different partition/sort keys
- **LSI** (Local Secondary Index): same partition key, different sort key

**Durability/HA**

- Multi-AZ by default within a region.

**Global tables**

- Multi-region replication for global low-latency reads/writes.

Exam “tell”:

- “**Need multi-region active-active NoSQL**” → DynamoDB Global Tables

**DynamoDB Streams**

- Change data capture; used for event-driven pipelines.

**DAX (DynamoDB Accelerator)**

- In-memory cache for read-heavy workloads.

Exam “tell”:

- “**Microsecond read latency for DynamoDB**” → DAX

**Common pitfalls tested**

- Hot partition problems (poor key design)
- Using DynamoDB for complex joins/queries (not ideal)

---

## 3. In-Memory Caching

### Amazon ElastiCache

Two engines:

#### Redis

- Rich data structures (lists/sets/sorted sets)
- Pub/sub
- Persistence options
- Replication and failover (depending on mode)

Use when:

- Session storage, leaderboards, caching with advanced structures

#### Memcached

- Simple key-value cache
- Typically easier for pure caching

Use when:

- You want a simple, highly-performant cache and don’t need Redis features

Exam “tell”:

- “**Need caching to reduce DB load**” → ElastiCache
- “**Session store / leaderboards / pubsub**” → Redis

---

## 4. Analytics / Warehousing

### Amazon Redshift

**What it is**
A data warehouse optimized for analytics over large structured datasets.

Use when:

- OLAP analytics, aggregations, BI dashboards

Common exam point:

- Redshift is for analytics (OLAP), not transactional OLTP.

---

## 5. Search and Log Analytics

### Amazon OpenSearch Service

**What it is**
Managed search and analytics engine (OpenSearch).

Use when:

- Full-text search
- Log analytics
- Near real-time indexing and queries

Common exam point:

- If you need “search” features on app data, OpenSearch is a common answer.

---

## 6. Specialty Databases

### 6.1 Amazon Neptune (Graph)

Use when:

- You need graph queries (relationships), e.g., social networks, recommendations, fraud rings.

### 6.2 Amazon DocumentDB (MongoDB-compatible)

Use when:

- You need MongoDB compatibility without self-managing MongoDB clusters.

### 6.3 Amazon Timestream (Time Series)

Use when:

- Time-stamped metrics/events at scale (IoT, observability).

### 6.4 Amazon QLDB (Ledger)

Use when:

- You need an immutable, cryptographically verifiable transaction log/audit trail.

---

## 7. Migration and Data Movement

### AWS DMS (Database Migration Service)

- Migrate relational DBs to AWS with minimal downtime.
- Often used with SCT (Schema Conversion Tool) when moving between engine types.

Exam “tell”:

- “**Migrate DB with minimal downtime**” → DMS

### CDC (Change Data Capture)

- Commonly referenced as “replicate changes while migrating”.

---

## 8. HA, DR, and Backups (Common Patterns)

### HA (within a region)

- **RDS Multi-AZ** for relational HA
- **Aurora** for advanced HA patterns
- **DynamoDB** is regional multi-AZ by default

### DR (cross-region)

- **Read replica in another region** (engine-dependent)
- **Aurora Global Database**
- **DynamoDB Global Tables**

### Backups

- RDS automated backups + snapshots
- DynamoDB point-in-time recovery (PITR) + backups

---

## 9. SAA-C03 “Tell” Phrases to Watch For

- “**ACID transactions, SQL joins**” → RDS/Aurora
- “**High availability with automatic failover**” → RDS Multi-AZ / Aurora
- “**Read-heavy relational workload**” → Read replicas
- “**Spiky traffic, serverless key-value store**” → DynamoDB on-demand
- “**Multi-region active-active NoSQL**” → DynamoDB Global Tables
- “**Reduce DB load with caching**” → ElastiCache
- “**Data warehouse / analytics**” → Redshift
- “**Full-text search / log analytics**” → OpenSearch
- “**Migrate with minimal downtime**” → DMS
- “**Lambda is opening too many DB connections**” → RDS Proxy
