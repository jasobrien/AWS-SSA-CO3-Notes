# AWS Solutions Architect Associate (SAA-C03) Practice Questions

20 original practice questions covering key SAA-C03 domains. Each question includes the correct answer and explanations for why other options are incorrect.

---

## Question 1 – High Availability Web App

**Domain:** Design Resilient Architectures

You are designing a highly available web application for a global audience. The application is stateless and runs on Amazon EC2 behind an Application Load Balancer (ALB). You must minimize downtime during deployments and avoid managing servers as much as possible.

Which combination of solutions is MOST appropriate?

A. Run EC2 instances in a single Availability Zone; use an ALB and rolling deployments
B. Use AWS Elastic Beanstalk with an ALB and deploy across multiple AZs
C. Use a Network Load Balancer with EC2 instances in multiple AZs and in-place updates
D. Use Amazon CloudFront in front of a single EC2 instance with EBS snapshots

**Correct answer:** B

**Why B is correct:**

- Elastic Beanstalk abstracts much of the server management, handles capacity provisioning, load balancing, and Auto Scaling.
- Deploying across multiple AZs increases availability, and Beanstalk supports blue/green or rolling deployments with minimal downtime.

**Why the other options are incorrect:**

- A: Single AZ is a single point of failure and does not meet high availability requirements.
- C: NLB is better for TCP or ultra-low-latency use cases; it doesn’t reduce server management overhead and relies on in-place updates.
- D: Single EC2 instance is a single point of failure; EBS snapshots are for backup, not high availability.

---

## Question 2 – Cost-Optimized Static Website

**Domain:** Design Cost-Optimized Architectures

A company wants to host a static marketing website (HTML, CSS, JavaScript) with minimal cost and global performance. They also want automatic scaling without managing servers.

Which architecture is MOST cost-effective and operationally simple?

A. EC2 in an Auto Scaling group behind an ALB
B. Amazon S3 static website hosting with Amazon CloudFront
C. Amazon EKS with managed node groups and NLB
D. AWS Elastic Beanstalk with t3.micro instances

**Correct answer:** B

**Why B is correct:**

- S3 static website hosting is serverless and low-cost.
- CloudFront provides global edge caching and improved performance without managing infrastructure.

**Why the other options are incorrect:**

- A: Requires managing EC2, load balancers, and scaling; overkill and more expensive for static content.
- C: EKS is a complex, managed Kubernetes service not justified for a simple static site.
- D: Beanstalk with EC2 instances is unnecessary for fully static content and increases cost and complexity.

---

## Question 3 – Data Durability and Access Patterns

**Domain:** Design Cost-Optimized Architectures

A team needs to store large volumes of infrequently accessed data that must be retained for 7 years for compliance. Occasionally, they must retrieve objects within minutes. Cost should be minimized.

Which S3 storage class is MOST appropriate?

A. S3 Standard
B. S3 One Zone-IA
C. S3 Glacier Instant Retrieval
D. S3 Glacier Flexible Retrieval

**Correct answer:** C

**Why C is correct:**

- S3 Glacier Instant Retrieval provides millisecond retrieval times, meeting the "within minutes" requirement while offering lower cost than S3 Standard for infrequently accessed data.
- Designed specifically for long-term archive data that needs immediate access when retrieved, perfect for compliance use cases.

**Why the other options are incorrect:**

- A: S3 Standard is significantly more expensive and optimized for frequent access, not cost-effective for 7-year retention with rare access.
- B: One Zone-IA stores data in a single AZ, which may not meet durability and compliance requirements for critical archives.
- D: Glacier Flexible Retrieval has retrieval times of 1-5 minutes (Expedited), 3-5 hours (Standard), or 5-12 hours (Bulk), which doesn't consistently meet "within minutes" requirement.

---

## Question 4 – RDS High Availability

**Domain:** Design Resilient Architectures

You run a production MySQL database on Amazon RDS. Management requires increased availability and automatic failover in case of an AZ outage.

What is the BEST way to meet this requirement?

A. Enable Multi-AZ deployment for the RDS instance
B. Create manual read replicas in another Region
C. Schedule regular automated snapshots to S3
D. Use Amazon DynamoDB instead of RDS

**Correct answer:** A

**Why A is correct:**

- Multi-AZ RDS provides synchronous replication to a standby in another AZ and automatic failover, directly addressing the high availability requirement.

**Why the other options are incorrect:**

- B: Read replicas are for read scaling and are asynchronous; they don’t provide automatic failover like Multi-AZ.
- C: Snapshots are for backup/restore, not real-time availability or automatic failover.
- D: Switching to DynamoDB is a major redesign and doesn’t inherently solve the requirement for the existing MySQL application.

---

## Question 5 – VPC Connectivity

**Domain:** Design High-Performing Architectures

A company needs secure connectivity between its on-premises data center and AWS VPC for a latency-sensitive application. They also require predictable bandwidth and private connectivity.

Which solution BEST meets these requirements?

A. AWS Site-to-Site VPN over the public Internet
B. AWS Direct Connect with a private virtual interface
C. VPC peering between on-premises and AWS
D. AWS Client VPN for all on-premises servers

**Correct answer:** B

**Why B is correct:**

- Direct Connect provides dedicated, private network connectivity with predictable performance and bandwidth, suitable for latency-sensitive workloads.

**Why the other options are incorrect:**

- A: VPN over the Internet is subject to variable latency and bandwidth.
- C: VPC peering is only between VPCs, not between on-premises and AWS.
- D: Client VPN is intended for users/clients, not as a primary link for data center servers.

---

## Question 6 – IAM Least Privilege

**Domain:** Design Secure Architectures

A developer needs to upload objects to a single S3 bucket but must not be able to delete any objects.

Which IAM policy is MOST appropriate?

A. Allow `s3:*` on the bucket
B. Allow `s3:PutObject` and `s3:ListBucket` on the bucket
C. Allow `s3:PutObject`, `s3:GetObject`, and `s3:DeleteObject` on the bucket
D. Allow `s3:PutObject` and `s3:*` on all resources

**Correct answer:** B

**Why B is correct:**

- Grants only the necessary permissions to upload (PutObject) and list (ListBucket) without delete, following least privilege.

**Why the other options are incorrect:**

- A: `s3:*` is overly permissive and includes delete operations.
- C: Explicitly grants `DeleteObject`, which is not allowed.
- D: Grants broad permissions on all resources, violating least privilege.

---

## Question 7 – Decoupling Components

**Domain:** Design Resilient Architectures

An application running on EC2 instances processes user uploads and sends them to a background worker for further processing. Spikes in uploads occasionally overwhelm the workers.

What is the BEST way to improve reliability and handle bursts of traffic?

A. Use an S3 bucket directly as the backend for workers
B. Use Amazon SQS between the web tier and worker tier
C. Increase the instance size of the worker EC2 instances
D. Use Amazon SNS to send notifications directly to workers

**Correct answer:** B

**Why B is correct:**

- Amazon SQS provides durable, scalable message queuing that decouples producers and consumers, smoothing out traffic spikes and improving reliability.

**Why the other options are incorrect:**

- A: S3 is object storage, not a queue; it doesn’t provide built-in ordering, retry, or visibility timeout semantics.
- C: Increasing instance size is a vertical scaling approach and doesn’t address burst handling or decoupling.
- D: SNS is pub/sub and pushes messages; it doesn’t offer the same decoupling, buffering, and pull-based processing as SQS.

---

## Question 8 – CloudFront and Security

**Domain:** Design Secure Architectures

You are using Amazon CloudFront in front of an S3 origin to serve static content. You want to restrict direct access to the S3 bucket so that only CloudFront can access it.

What should you do?

A. Make the S3 bucket public-read and rely on CloudFront for caching
B. Use an S3 bucket policy that only allows access from CloudFront origin access identity (OAI) or OAC
C. Enable versioning on the S3 bucket
D. Put the S3 bucket in a private subnet

**Correct answer:** B

**Why B is correct:**

- Configuring an origin access identity (OAI) or Origin Access Control (OAC) and updating the S3 bucket policy to allow only that principal ensures only CloudFront can access the bucket.

**Why the other options are incorrect:**

- A: Making the bucket public defeats the purpose of restricting direct access.
- C: Versioning is for object recovery and lifecycle management, not access control.
- D: S3 is a regional service, not placed in a VPC subnet; this option is not valid.

---

## Question 9 – Serverless Data Processing

**Domain:** Design High-Performing Architectures

A team wants to trigger data processing whenever a new object is uploaded to an S3 bucket. The processing must scale automatically and they don’t want to manage servers.

Which architecture is MOST appropriate?

A. S3 event notifications triggering an AWS Lambda function
B. S3 event notifications triggering an EC2 instance
C. S3 event notifications triggering an RDS stored procedure
D. Poll the S3 bucket periodically from an on-premises server

**Correct answer:** A

**Why A is correct:**

- S3 can directly invoke Lambda on object uploads, providing serverless, event-driven processing that scales automatically.

**Why the other options are incorrect:**

- B: EC2-based processing adds server management overhead and doesn’t scale as seamlessly.
- C: RDS cannot be triggered directly from S3 events.
- D: Polling from on-premises adds latency, complexity, and doesn’t scale well.

---

## Question 10 – Multi-Region Disaster Recovery

**Domain:** Design Resilient Architectures

A critical application must be able to fail over to another AWS Region in case of a regional outage. The RTO is 1 hour and RPO is 15 minutes. Cost should be minimized while meeting these targets.

Which strategy is MOST appropriate?

A. Backup and restore using daily S3 backups
B. Pilot light with minimal infrastructure in the secondary Region and regular data replication
C. Active-active multi-Region with traffic distributed by Route 53 latency routing
D. Store data only in S3 with cross-Region replication and recreate the app when needed

**Correct answer:** B

**Why B is correct:**

- Pilot light maintains a minimal version of the environment in the secondary Region with ongoing data replication; it supports reasonably low RTO/RPO at lower cost than full active-active.

**Why the other options are incorrect:**

- A: Daily backups won’t meet a 1-hour RTO and 15-minute RPO.
- C: Active-active is more complex and costly than necessary; while valid, it’s usually chosen when continuous active presence in both Regions is required, not just DR.
- D: Recreating the app on demand from S3 would likely exceed the 1-hour RTO.

---

## Question 11 – Aurora vs RDS

**Domain:** Design High-Performing Architectures

You are choosing between Amazon RDS for MySQL and Amazon Aurora MySQL-Compatible Edition. The application requires high throughput, automatic scaling of read capacity, and fast recovery.

Which choice is MOST appropriate?

A. RDS for MySQL with a single AZ deployment
B. RDS for MySQL with multiple read replicas
C. Amazon Aurora MySQL-Compatible cluster with reader endpoints
D. Self-managed MySQL on EC2

**Correct answer:** C

**Why C is correct:**

- Aurora is designed for high throughput, fast recovery, and can automatically scale read capacity using read replicas behind a reader endpoint.

**Why the other options are incorrect:**

- A: Single AZ RDS doesn’t provide high availability or scalable read capacity.
- B: RDS read replicas provide scaling, but management and recovery are not as streamlined as Aurora, and performance may be lower.
- D: Self-managed MySQL on EC2 increases operational burden and complexity.

---

## Question 12 – Application Load Balancer Features

**Domain:** Design High-Performing Architectures

Which feature is supported by an Application Load Balancer (ALB) but NOT by a Network Load Balancer (NLB)?

A. Support for TCP traffic
B. Path-based routing (e.g., /api vs /images)
C. Static IP addresses
D. High performance at ultra-low latency

**Correct answer:** B

**Why B is correct:**

- ALBs operate at Layer 7 and support advanced HTTP features like path-based and host-based routing.

**Why the other options are incorrect:**

- A: NLB supports TCP; ALB supports HTTP/HTTPS (and HTTP/2/WebSockets).
- C: NLB supports static IP addresses; ALB uses DNS, not fixed IPs by default.
- D: NLB is optimized for ultra-low latency and high performance at Layer 4.

---

## Question 13 – S3 Encryption Requirements

**Domain:** Design Secure Architectures

A company must encrypt all objects stored in S3 and wants AWS to manage the encryption keys. They also want the option to audit key usage.

Which encryption option is MOST appropriate?

A. SSE-S3 (Amazon S3-managed keys)
B. SSE-KMS (AWS KMS-managed keys)
C. Client-side encryption only
D. No encryption is needed because S3 is secure by default

**Correct answer:** B

**Why B is correct:**

- SSE-KMS uses KMS-managed keys and provides detailed CloudTrail audit logs for key usage.

**Why the other options are incorrect:**

- A: SSE-S3 encrypts data at rest but does not provide per-request key usage auditing.
- C: Client-side encryption may be valid but shifts key management to the customer and complicates auditing.
- D: S3 is secure but data is not encrypted at rest by default without enabling an encryption option.

---

## Question 14 – Auto Scaling Policies

**Domain:** Design High-Performing Architectures

You have an Auto Scaling group for a web application. You want to keep CPU utilization around 50% across the group while automatically scaling out and in.

Which scaling policy type is MOST appropriate?

A. Scheduled scaling
B. Step scaling
C. Simple scaling
D. Target tracking scaling

**Correct answer:** D

**Why D is correct:**

- Target tracking scaling adjusts capacity to maintain a specified metric (like average CPU utilization) at a target value.

**Why the other options are incorrect:**

- A: Scheduled scaling is based on time, not real-time load.
- B: Step scaling reacts to threshold breaches with predefined steps but doesn’t maintain a specific target value easily.
- C: Simple scaling is an older model that responds to alarms but does not maintain a continuous target level.

---

## Question 15 – VPC Subnet Design

**Domain:** Design Secure Architectures

You are designing a VPC for a three-tier web application. You must expose the web tier to the Internet but keep the application and database tiers private.

Which design is MOST appropriate?

A. Place all tiers in public subnets with security groups
B. Place the web tier in public subnets, app and DB tiers in private subnets
C. Place all tiers in private subnets with no Internet access
D. Place web and app tiers in public subnets, DB tier in private subnets

**Correct answer:** B

**Why B is correct:**

- Public subnets for the web tier with an Internet Gateway, and private subnets for app and DB tiers, is a standard secure 3-tier architecture.

**Why the other options are incorrect:**

- A: Exposes the app and DB tiers directly to the Internet, increasing attack surface.
- C: Web tier can’t serve Internet traffic without some public access.
- D: App tier should also be private and only accessible from the web tier, not directly from the Internet.

---

## Question 16 – Logging and Monitoring

**Domain:** Design Resilient Architectures

A company wants a centralized solution to collect logs from EC2 instances, Lambda, and RDS, and set alarms on specific metrics and error patterns.

Which AWS service combination is MOST appropriate?

A. CloudTrail and S3 only
B. CloudWatch Logs, CloudWatch Metrics, and CloudWatch Alarms
C. AWS Config and CloudTrail
D. Amazon Inspector and AWS Config

**Correct answer:** B

**Why B is correct:**

- CloudWatch Logs aggregates logs, CloudWatch Metrics provides metrics, and CloudWatch Alarms can trigger notifications/actions based on metric thresholds or patterns.

**Why the other options are incorrect:**

- A: CloudTrail focuses on API activity, not general application logs or metrics.
- C: AWS Config tracks configuration changes, not general log/metric aggregation and alarming.
- D: Inspector is for security assessments, not centralized logging/monitoring.

---

## Question 17 – Cost Optimization with EC2

**Domain:** Design Cost-Optimized Architectures

A batch processing job runs every night for 3 hours and can tolerate interruptions. Cost must be minimized.

Which EC2 purchasing option is MOST appropriate?

A. On-Demand Instances
B. Reserved Instances
C. Spot Instances
D. Dedicated Hosts

**Correct answer:** C

**Why C is correct:**

- Spot Instances offer significant discounts for workloads that can tolerate interruption, which fits a nightly batch job.

**Why the other options are incorrect:**

- A: On-Demand is more expensive for this flexible workload.
- B: Reserved Instances are best for steady, long-term usage, not sporadic nightly jobs.
- D: Dedicated Hosts are for compliance/licensing use cases and are more expensive.

---

## Question 18 – Database Caching

**Domain:** Design High-Performing Architectures

A read-heavy application experiences high latency when querying an RDS database. You want to reduce latency and offload reads from the database.

Which solution is MOST appropriate?

A. Add more IOPS to the RDS instance
B. Add Amazon ElastiCache (Redis or Memcached) in front of RDS
C. Migrate RDS to a larger instance type only
D. Use S3 as a cache for database queries

**Correct answer:** B

**Why B is correct:**

- ElastiCache provides an in-memory cache layer that can greatly reduce read latency and number of DB queries.

**Why the other options are incorrect:**

- A: More IOPS may help storage performance but doesn’t directly address the caching use case.
- C: Scaling up the instance is a vertical scaling approach and may be more expensive and less effective than caching.
- D: S3 is object storage and not suitable as a low-latency cache for dynamic query results.

---

## Question 19 – API Gateway and Lambda

**Domain:** Design High-Performing Architectures

You are building a serverless REST API that must scale automatically and integrate with multiple backend Lambda functions.

Which architecture is MOST appropriate?

A. API Gateway REST API integrated with AWS Lambda functions
B. ALB forwarding requests directly to Lambda functions
C. EC2 instances behind an NLB
D. Lambda functions invoked via S3 events only

**Correct answer:** A

**Why A is correct:**

- API Gateway provides REST API capabilities, request validation, throttling, and can directly integrate with multiple Lambda backends in a serverless architecture.

**Why the other options are incorrect:**

- B: ALB can integrate with Lambda, but API Gateway is the more feature-rich, purpose-built solution for REST APIs.
- C: Requires managing EC2 infrastructure and is not serverless.
- D: S3 event triggers don’t provide a general-purpose REST API.

---

## Question 20 – Cross-Account Access

**Domain:** Design Secure Architectures

A security team in a central AWS account needs read-only access to CloudTrail logs stored in an S3 bucket in another AWS account.

What is the MOST secure and manageable way to grant this access?

A. Make the S3 bucket public-read
B. Use an S3 bucket policy to allow access to an IAM role in the security account, and have users assume that role
C. Create IAM users in the logging account for each security engineer
D. Copy logs daily to the security account’s S3 bucket manually

**Correct answer:** B

**Why B is correct:**

- Granting cross-account access via an S3 bucket policy to a role in the security account, and using role assumption, is secure, auditable, and scalable.

**Why the other options are incorrect:**

- A: Public-read is insecure and violates best practices.
- C: Managing individual IAM users across accounts is less scalable and secure than using roles.
- D: Manual copies increase operational overhead and risk inconsistent access to logs.

---

These questions are intended as practice only and are not official AWS exam questions.
