# AWS Analytics & Data Lake (SAA-C03) — High-Yield Notes

This chapter covers the analytics/data-lake concepts that show up frequently in SAA-C03 scenario questions: **S3 data lakes**, **Athena**, **Glue**, **Lake Formation**, and the most common “which service should I pick?” decision points.

---

## Quick Decision Guide

- **Need to query data in S3 using SQL with no servers?** → **Athena**
- **Need an ETL service + schedule jobs + transform data for analytics?** → **AWS Glue**
- **Need a central schema/metadata catalog for data in S3?** → **Glue Data Catalog**
- **Need fine-grained data lake permissions (table/column/row) centrally?** → **Lake Formation**
- **Need a data warehouse for high-concurrency BI and complex analytics?** → **Redshift**
- **Need to stream data into S3/Redshift/OpenSearch with minimal ops?** → **Kinesis Data Firehose**
- **Need big-data clusters you control (Spark/Hadoop) and can tune?** → **EMR** (less common, but still exam-relevant)

---

## 1. S3 as a Data Lake (Common Exam Framing)

### What it means

A “data lake” in SAA terms usually means:

- Raw and curated data stored in **S3**
- Querying via **Athena** or loading into **Redshift**
- Metadata in a **catalog** (often Glue)
- Governance via **Lake Formation** when permissions are emphasized

### Common exam “tell”

- “Store large amounts of structured/semi-structured data cheaply and query it on demand” → S3 + Athena

---

## 2. Amazon Athena

### What it is

Serverless interactive query service that uses SQL to query data in **S3** (via Presto/Trino under the hood).

### When it’s a great answer

- Ad-hoc queries against logs, CSV/JSON/Parquet in S3
- “No servers to manage” analytics
- You want to pay only when you query

### Performance and cost levers (high-yield)

Athena cost is driven largely by **bytes scanned**.

To reduce cost / increase performance:

- **Partition data** (by date, region, customer, etc.)
- Use **columnar formats**: **Parquet** / **ORC**
- Use compression
- Select only required columns (don’t `SELECT *`)

Exam “tell”:

- “Athena queries are expensive / scan too much data” → partition + Parquet/ORC + compression

### Athena vs Redshift (the common comparison)

- **Athena**: query-in-place on S3 (serverless, pay per query)
- **Redshift**: data warehouse (more tuning, better for consistent high concurrency / complex analytics)

Exam “tell”:

- “BI dashboards with many concurrent users and consistent performance” → Redshift
- “Occasional queries on large S3 datasets” → Athena

---

## 3. AWS Glue

### What it is

Managed ETL service.

Core components you see in SAA:

- **Glue Data Catalog**: central metadata repository (tables, schemas) used by Athena/Redshift Spectrum/EMR
- **Crawlers**: scan data in S3 and create/update catalog tables
- **Jobs**: ETL transformations (often Spark-based)

### When it’s the answer

- “Transform raw data in S3 into curated/analytics-ready format”
- “Scheduled ETL job”
- “Automatically discover schema from S3” → crawler

Exam “tell”:

- “Need to catalog data and make it queryable” → Glue Data Catalog (+ crawler)

---

## 4. Lake Formation

### What it is

A governance layer for data lakes (primarily S3 + Glue catalog) that centralizes access control.

What it adds (exam-level):

- Central permissions on databases/tables/columns (and sometimes row-level patterns)
- Delegated administration for data access
- Integrates with Glue Data Catalog

When it’s the answer:

- “Fine-grained access control for data in S3 (table/column) across many teams/accounts”
- “Central data lake governance”

Exam “tell”:

- “Different teams must only see certain columns/records” → Lake Formation permissions

---

## 5. Redshift (Context)

Redshift is already covered in the Databases chapter; the data-lake exam comparison is usually:

- Use **Redshift** for a warehouse (performance and concurrency)
- Use **Athena** for serverless query-in-place on S3

A common hybrid pattern:

- S3 as the lake → Athena for ad-hoc → Redshift for curated warehouse/BI

---

## 6. Streaming Data into the Lake (Common Pattern)

If the question says “stream events/logs continuously into S3 with minimal ops”:

- **Kinesis Data Firehose → S3** is a frequent best answer

If the question emphasizes replay and multiple consumers:

- **Kinesis Data Streams** (then consumers write to S3/Redshift/etc.)

---

## 7. SAA-C03 “Tell” Phrases to Watch For

- “**Query files in S3 using SQL**” → Athena
- “**Reduce Athena cost / scanning too much data**” → partitioning + Parquet/ORC + compression
- “**Discover schema / catalog datasets in S3**” → Glue crawler + Glue Data Catalog
- “**ETL transform data in S3**” → Glue jobs
- “**Fine-grained permissions on S3 data lake tables/columns**” → Lake Formation
- “**High concurrency BI dashboards / consistent analytics performance**” → Redshift
- “**Stream data into S3 with minimal management**” → Kinesis Data Firehose
