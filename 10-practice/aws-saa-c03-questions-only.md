# AWS Solutions Architect Associate (SAA-C03) – Practice Questions (Questions Only)

Use this file to practice. Answers and explanations are in `aws-saa-c03-practice-qanda.md`.

---

1. **High Availability Web App**  
    **Domain:** Design Resilient Architectures  
   You are designing a highly available web application for a global audience. The application is stateless and runs on Amazon EC2 behind an Application Load Balancer (ALB). You must minimize downtime during deployments and avoid managing servers as much as possible.  
   Which combination of solutions is MOST appropriate?

A. Run EC2 instances in a single Availability Zone; use an ALB and rolling deployments  
B. Use AWS Elastic Beanstalk with an ALB and deploy across multiple AZs  
C. Use a Network Load Balancer with EC2 instances in multiple AZs and in-place updates  
D. Use Amazon CloudFront in front of a single EC2 instance with EBS snapshots

---

2. **Cost-Optimized Static Website**  
    **Domain:** Design Cost-Optimized Architectures  
   A company wants to host a static marketing website (HTML, CSS, JavaScript) with minimal cost and global performance. They also want automatic scaling without managing servers.  
   Which architecture is MOST cost-effective and operationally simple?

A. EC2 in an Auto Scaling group behind an ALB  
B. Amazon S3 static website hosting with Amazon CloudFront  
C. Amazon EKS with managed node groups and NLB  
D. AWS Elastic Beanstalk with t3.micro instances

---

3. **Data Durability and Access Patterns**  
    **Domain:** Design Cost-Optimized Architectures  
   A team needs to store large volumes of infrequently accessed data that must be retained for 7 years for compliance. Occasionally, they must retrieve objects within minutes. Cost should be minimized.  
   Which S3 storage class is MOST appropriate?

A. S3 Standard  
B. S3 One Zone-IA  
C. S3 Glacier Instant Retrieval  
D. S3 Glacier Flexible Retrieval

---

4. **RDS High Availability**  
    **Domain:** Design Resilient Architectures  
   You run a production MySQL database on Amazon RDS. Management requires increased availability and automatic failover in case of an AZ outage.  
   What is the BEST way to meet this requirement?

A. Enable Multi-AZ deployment for the RDS instance  
B. Create manual read replicas in another Region  
C. Schedule regular automated snapshots to S3  
D. Use Amazon DynamoDB instead of RDS

---

5. **VPC Connectivity**  
    **Domain:** Design High-Performing Architectures  
   A company needs secure connectivity between its on-premises data center and AWS VPC for a latency-sensitive application. They also require predictable bandwidth and private connectivity.  
   Which solution BEST meets these requirements?

A. AWS Site-to-Site VPN over the public Internet  
B. AWS Direct Connect with a private virtual interface  
C. VPC peering between on-premises and AWS  
D. AWS Client VPN for all on-premises servers

---

6. **IAM Least Privilege**  
    **Domain:** Design Secure Architectures  
   A developer needs to upload objects to a single S3 bucket but must not be able to delete any objects.  
   Which IAM policy is MOST appropriate?

A. Allow `s3:*` on the bucket  
B. Allow `s3:PutObject` and `s3:ListBucket` on the bucket  
C. Allow `s3:PutObject`, `s3:GetObject`, and `s3:DeleteObject` on the bucket  
D. Allow `s3:PutObject` and `s3:*` on all resources

---

7. **Decoupling Components**  
    **Domain:** Design Resilient Architectures  
   An application running on EC2 instances processes user uploads and sends them to a background worker for further processing. Spikes in uploads occasionally overwhelm the workers.  
   What is the BEST way to improve reliability and handle bursts of traffic?

A. Use an S3 bucket directly as the backend for workers  
B. Use Amazon SQS between the web tier and worker tier  
C. Increase the instance size of the worker EC2 instances  
D. Use Amazon SNS to send notifications directly to workers

---

8. **CloudFront and Security**  
    **Domain:** Design Secure Architectures  
   You are using Amazon CloudFront in front of an S3 origin to serve static content. You want to restrict direct access to the S3 bucket so that only CloudFront can access it.  
   What should you do?

A. Make the S3 bucket public-read and rely on CloudFront for caching  
B. Use an S3 bucket policy that only allows access from CloudFront origin access identity (OAI) or OAC  
C. Enable versioning on the S3 bucket  
D. Put the S3 bucket in a private subnet

---

9. **Serverless Data Processing**  
    **Domain:** Design High-Performing Architectures  
   A team wants to trigger data processing whenever a new object is uploaded to an S3 bucket. The processing must scale automatically and they don’t want to manage servers.  
   Which architecture is MOST appropriate?

A. S3 event notifications triggering an AWS Lambda function  
B. S3 event notifications triggering an EC2 instance  
C. S3 event notifications triggering an RDS stored procedure  
D. Poll the S3 bucket periodically from an on-premises server

---

10. **Multi-Region Disaster Recovery**  
    **Domain:** Design Resilient Architectures  
    A critical application must be able to fail over to another AWS Region in case of a regional outage. The RTO is 1 hour and RPO is 15 minutes. Cost should be minimized while meeting these targets.  
    Which strategy is MOST appropriate?

A. Backup and restore using daily S3 backups  
B. Pilot light with minimal infrastructure in the secondary Region and regular data replication  
C. Active-active multi-Region with traffic distributed by Route 53 latency routing  
D. Store data only in S3 with cross-Region replication and recreate the app when needed

---

11. **Aurora vs RDS**  
    **Domain:** Design High-Performing Architectures  
    You are choosing between Amazon RDS for MySQL and Amazon Aurora MySQL-Compatible Edition. The application requires high throughput, automatic scaling of read capacity, and fast recovery.  
    Which choice is MOST appropriate?

A. RDS for MySQL with a single AZ deployment  
B. RDS for MySQL with multiple read replicas  
C. Amazon Aurora MySQL-Compatible cluster with reader endpoints  
D. Self-managed MySQL on EC2

---

12. **Application Load Balancer Features**  
    **Domain:** Design High-Performing Architectures  
    Which feature is supported by an Application Load Balancer (ALB) but NOT by a Network Load Balancer (NLB)?

A. Support for TCP traffic  
B. Path-based routing (e.g., /api vs /images)  
C. Static IP addresses  
D. High performance at ultra-low latency

---

13. **S3 Encryption Requirements**  
    **Domain:** Design Secure Architectures  
    A company must encrypt all objects stored in S3 and wants AWS to manage the encryption keys. They also want the option to audit key usage.  
    Which encryption option is MOST appropriate?

A. SSE-S3 (Amazon S3-managed keys)  
B. SSE-KMS (AWS KMS-managed keys)  
C. Client-side encryption only  
D. No encryption is needed because S3 is secure by default

---

14. **Auto Scaling Policies**  
    **Domain:** Design High-Performing Architectures  
    You have an Auto Scaling group for a web application. You want to keep CPU utilization around 50% across the group while automatically scaling out and in.  
    Which scaling policy type is MOST appropriate?

A. Scheduled scaling  
B. Step scaling  
C. Simple scaling  
D. Target tracking scaling

---

15. **VPC Subnet Design**  
    **Domain:** Design Secure Architectures  
    You are designing a VPC for a three-tier web application. You must expose the web tier to the Internet but keep the application and database tiers private.  
    Which design is MOST appropriate?

A. Place all tiers in public subnets with security groups  
B. Place the web tier in public subnets, app and DB tiers in private subnets  
C. Place all tiers in private subnets with no Internet access  
D. Place web and app tiers in public subnets, DB tier in private subnets

---

16. **Logging and Monitoring**  
    **Domain:** Design Resilient Architectures  
    A company wants a centralized solution to collect logs from EC2 instances, Lambda, and RDS, and set alarms on specific metrics and error patterns.  
    Which AWS service combination is MOST appropriate?

A. CloudTrail and S3 only  
B. CloudWatch Logs, CloudWatch Metrics, and CloudWatch Alarms  
C. AWS Config and CloudTrail  
D. Amazon Inspector and AWS Config

---

17. **Cost Optimization with EC2**  
    **Domain:** Design Cost-Optimized Architectures  
    A batch processing job runs every night for 3 hours and can tolerate interruptions. Cost must be minimized.  
    Which EC2 purchasing option is MOST appropriate?

A. On-Demand Instances  
B. Reserved Instances  
C. Spot Instances  
D. Dedicated Hosts

---

18. **Database Caching**  
    **Domain:** Design High-Performing Architectures  
    A read-heavy application experiences high latency when querying an RDS database. You want to reduce latency and offload reads from the database.  
    Which solution is MOST appropriate?

A. Add more IOPS to the RDS instance  
B. Add Amazon ElastiCache (Redis or Memcached) in front of RDS  
C. Migrate RDS to a larger instance type only  
D. Use S3 as a cache for database queries

---

19. **API Gateway and Lambda**  
    **Domain:** Design High-Performing Architectures  
    You are building a serverless REST API that must scale automatically and integrate with multiple backend Lambda functions.  
    Which architecture is MOST appropriate?

A. API Gateway REST API integrated with AWS Lambda functions  
B. ALB forwarding requests directly to Lambda functions  
C. EC2 instances behind an NLB  
D. Lambda functions invoked via S3 events only

---

20. **Cross-Account Access**  
    **Domain:** Design Secure Architectures  
    A security team in a central AWS account needs read-only access to CloudTrail logs stored in an S3 bucket in another AWS account.  
    What is the MOST secure and manageable way to grant this access?

A. Make the S3 bucket public-read  
B. Use an S3 bucket policy to allow access to an IAM role in the security account, and have users assume that role  
C. Create IAM users in the logging account for each security engineer  
D. Copy logs daily to the security account’s S3 bucket manually
