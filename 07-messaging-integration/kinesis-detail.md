# AWS Kinesis Services Overview

AWS **Kinesis** is a suite of services for **real-time streaming data ingestion, processing, and analytics**. It is designed for applications that need to handle data continuously with low latency.

---

## What is Kinesis used for?

Common use cases include:

* Clickstream and user activity
* Application logs and metrics
* IoT sensor data
* Financial transactions
* Telemetry and monitoring
* Video streaming and analysis

---

## The Kinesis Service Family

| Service                        | Purpose                                               |
| ------------------------------ | ----------------------------------------------------- |
| **Kinesis Data Streams (KDS)** | Custom real-time data streams with multiple consumers |
| **Kinesis Data Firehose**      | Fully managed streaming data delivery                 |
| **Kinesis Data Analytics**     | Real-time analytics using SQL or Apache Flink         |
| **Kinesis Video Streams**      | Streaming video and time-encoded media                |

---

## Kinesis Data Streams (KDS)

### Description

A **low-level, highly configurable real-time streaming service**. You manage throughput, scaling, and consumers.

### Core Concepts

| Term          | Description                    |
| ------------- | ------------------------------ |
| Stream        | The data pipeline              |
| Shard         | Unit of capacity               |
| Record        | A single data item             |
| Partition key | Determines shard placement     |
| Consumer      | Application reading the stream |

### Throughput per shard

* 1 MB/sec writes
* 2 MB/sec reads
* 1,000 records/sec

### Retention

* Default: 24 hours
* Extended: up to 365 days

### Typical consumers

* AWS Lambda
* EC2 / ECS applications
* Kinesis Data Analytics

### When to use

* Multiple consumers required
* Data replay needed
* Custom stream processing logic

---

## Kinesis Data Firehose

### Description

A **fully managed service** for delivering streaming data to destinations with minimal setup.

### Key characteristics

* No shards to manage
* Automatic scaling
* Near real-time delivery (buffered)
* Optional Lambda transformation

### Supported destinations

* Amazon S3
* Amazon Redshift
* Amazon OpenSearch
* Splunk
* HTTP endpoints

### When to use

* Log ingestion
* Streaming data into S3 or analytics systems
* Simple pipelines without custom consumers

---

## Kinesis Data Analytics (KDA)

### Description

A service for **real-time analytics on streaming data**.

### Processing options

* SQL (simpler use cases)
* Apache Flink (advanced streaming analytics)

### Input sources

* Kinesis Data Streams
* Kinesis Data Firehose

### Output targets

* Kinesis Data Streams
* Firehose
* S3
* OpenSearch

### Typical use cases

* Real-time aggregations
* Fraud detection
* Sliding windows
* Anomaly detection

---

## Kinesis Video Streams (KVS)

### Description

A streaming service for **video, audio, and frame-based data**.

### Use cases

* Security and surveillance cameras
* Dashcams
* Machine vision
* Media playback
* Facial recognition

### Integrations

* Amazon Rekognition Video
* Amazon S3
* Custom ML pipelines

---

## Service Comparison

| Feature            | Data Streams | Firehose       | Data Analytics | Video Streams |
| ------------------ | ------------ | -------------- | -------------- | ------------- |
| Real-time          | Yes          | Near real-time | Yes            | Yes           |
| Shard management   | Yes          | No             | N/A            | No            |
| Replay data        | Yes          | No             | Depends        | Yes           |
| Multiple consumers | Yes          | No             | N/A            | Yes           |
| SQL analytics      | No           | No             | Yes            | No            |
| Video support      | No           | No             | No             | Yes           |

---

## Exam-Oriented Decision Guide

* **Real-time processing with multiple consumers** → Kinesis Data Streams
* **Stream data directly into S3 or analytics services** → Kinesis Data Firehose
* **SQL analytics on streaming data** → Kinesis Data Analytics
* **Live video ingestion and processing** → Kinesis Video Streams

---

## One-line Memory Aid

> **Streams = control**, **Firehose = delivery**, **Analytics = SQL/Flink**, **Video = media**
