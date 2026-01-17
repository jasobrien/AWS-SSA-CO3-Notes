# Web Application Architectures on AWS: EC2, ECS Fargate, and Database Options

This document explains common AWS architectures for a web application with web app and database tiers, including autoscaling, logging, and alerting. It compares solutions using EC2, ECS Fargate, and different database types, and discusses their pros and cons.

---

## 1. EC2-based Architecture

**Overview:**

**Diagram:**

```
[User] → [ELB] → [EC2 Auto Scaling Group] → [RDS Database]
```

**Pros:**

- Full control over OS and runtime.
- Can run any software or custom agents.
- Mature, well-documented pattern.

**Cons:**

- More operational overhead (patching, scaling, recovery).
- Slower scaling compared to containers.
- Must manage AMIs and instance health.

---

## 2. ECS Fargate-based Architecture

**Overview:**

**Diagram:**

```
[User] → [ALB] → [ECS Fargate Service] → [RDS/Aurora/DynamoDB]
```

**Pros:**

- No server management (serverless containers).
- Fast, granular scaling (per task).
- Easy blue/green deployments.
- Integrates well with CI/CD.

**Cons:**

- Less control over underlying OS.
- Cold start latency for new tasks.
- Fargate cost can be higher for steady, predictable workloads.

---

## 3. Database Options

### a. Amazon RDS (MySQL/PostgreSQL/Aurora)

- Managed relational database.
- Automated backups, patching, and scaling.
- Multi-AZ and read replicas for HA and scaling.

**Pros:**

- Familiar SQL interface.
- Easy migration from on-premises databases.

**Cons:**

- Scaling write throughput can be challenging.
- More expensive than self-managed DBs for some workloads.

### b. Amazon DynamoDB

- Fully managed NoSQL database.
- Serverless, auto-scaling, and highly available.

**Pros:**

- Virtually unlimited scale.
- No server management.
- Millisecond latency at scale.

**Cons:**

- No SQL joins or complex queries.
- Data modeling is different from relational DBs.

---

## 4. Autoscaling, Logging, and Alerts

- **Autoscaling:**

  - EC2: Auto Scaling Groups based on CPU, network, or custom metrics.
  - ECS Fargate: Service Auto Scaling based on CPU, memory, or custom CloudWatch metrics.
  - RDS: Read replicas and Aurora Serverless for scaling.
  - DynamoDB: On-demand or provisioned capacity with auto scaling.

- **Example: EC2 Auto Scaling Group Configuration (CloudFormation YAML):**

```yaml
AutoScalingGroup:
  Type: AWS::AutoScaling::AutoScalingGroup
  Properties:
    MinSize: "2"
    MaxSize: "10"
    DesiredCapacity: "4"
    LaunchTemplate:
      LaunchTemplateId: !Ref MyLaunchTemplate
      Version: !GetAtt MyLaunchTemplate.LatestVersionNumber
    TargetGroupARNs:
      - !Ref MyTargetGroup
    VPCZoneIdentifier:
      - subnet-123456
      - subnet-654321
    MetricsCollection:
      - Granularity: "1Minute"
    Tags:
      - Key: Name
        Value: webapp-ec2-instance
        PropagateAtLaunch: true
```

- **Example: ECS Fargate Service Auto Scaling (CloudFormation YAML):**

```yaml
ServiceScalingTarget:
  Type: AWS::ApplicationAutoScaling::ScalableTarget
  Properties:
    MaxCapacity: 10
    MinCapacity: 2
    ResourceId: service/myCluster/myService
    RoleARN: !GetAtt MyScalingRole.Arn
    ScalableDimension: ecs:service:DesiredCount
    ServiceNamespace: ecs

ServiceScalingPolicy:
  Type: AWS::ApplicationAutoScaling::ScalingPolicy
  Properties:
    PolicyName: cpu-scaling-policy
    PolicyType: TargetTrackingScaling
    ScalingTargetId: !Ref ServiceScalingTarget
    TargetTrackingScalingPolicyConfiguration:
      TargetValue: 60.0
      PredefinedMetricSpecification:
        PredefinedMetricType: ECSServiceAverageCPUUtilization
      ScaleInCooldown: 60
      ScaleOutCooldown: 60
```

- **Logging:**

  - Application and access logs to CloudWatch Logs.
  - Database logs to CloudWatch (RDS/Aurora) or CloudTrail (DynamoDB).

- **Alerts:**
  - CloudWatch Alarms for high CPU, memory, error rates, slow queries, etc.
  - SNS notifications or automated remediation actions.

---

## 5. Summary Table

| Solution               | Pros                           | Cons                         |
| ---------------------- | ------------------------------ | ---------------------------- |
| EC2 + RDS              | Full control, mature, flexible | More ops, slower scaling     |
| ECS Fargate + RDS      | No server mgmt, fast scaling   | Less OS control, cold starts |
| ECS Fargate + DynamoDB | Serverless, unlimited scale    | No SQL, new data modeling    |

---

## 6. Choosing the Right Solution

- **EC2**: Best for legacy apps, custom OS needs, or when you need full control.
- **ECS Fargate**: Best for modern, containerized apps needing fast scaling and minimal ops.
- **RDS**: Use for relational data and SQL workloads.
- **DynamoDB**: Use for high-scale, NoSQL workloads with simple access patterns.

---

## 7. Security Group Table Examples

Below are example security group rules for a typical 2-tier web application:

| Security Group | Source      | Protocol | Port Range | Description                  |
| -------------- | ----------- | -------- | ---------- | ---------------------------- |
| WebSG          | 0.0.0.0/0   | TCP      | 80, 443    | Allow HTTP/HTTPS from public |
| WebSG          | 10.0.0.0/16 | TCP      | 22         | Allow SSH from corp network  |
| DB-SG          | WebSG       | TCP      | 3306       | Allow MySQL from web tier    |
| DB-SG          | 10.0.0.0/16 | TCP      | 5432       | Allow PostgreSQL from corp   |

**Notes:**

- WebSG: Security group for EC2 or ECS tasks in the web/app tier.
- DB-SG: Security group for RDS or database tier.
- Adjust port numbers for your DB engine (e.g., 1433 for SQL Server).

---

**All architectures should implement autoscaling, centralized logging, and alerting for production readiness.**

---

## 8. Required IAM Roles and Permissions

Below are typical AWS IAM roles and permissions required for each solution. Always follow the principle of least privilege and tailor policies to your environment.

### EC2-based Solution

- **EC2 Instance Role:**
  - `AmazonSSMManagedInstanceCore` (for SSM access/management)
  - `CloudWatchAgentServerPolicy` (for logging/metrics)
  - Custom policy for S3, Secrets Manager, Parameter Store, etc., as needed
- **RDS Access:**
  - Application instances need DB credentials (use Secrets Manager or Parameter Store)
  - No direct IAM access to RDS unless using IAM DB authentication

### ECS Fargate-based Solution

- **Task Execution Role:**
  - `AmazonECSTaskExecutionRolePolicy` (pull images, write logs)
- **Task Role:**
  - Custom policy for S3, DynamoDB, SQS, SNS, Secrets Manager, Parameter Store, etc., as needed by the app
- **RDS Access:**
  - As above, use Secrets Manager/Parameter Store for DB credentials
- **DynamoDB Access:**
  - Grant `dynamodb:PutItem`, `dynamodb:GetItem`, etc., as needed

### Autoscaling and Monitoring

- **Auto Scaling:**
  - `AutoScalingFullAccess` (for management, not needed for runtime)
- **CloudWatch Alarms/Logs:**
  - `CloudWatchFullAccess` (for management)
  - Runtime roles need only `PutMetricData`, `CreateLogStream`, `PutLogEvents`

### Example: ECS Task Execution Role Policy (JSON)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

**Note:**

- Always scope resources and actions as tightly as possible.
- Use managed policies where appropriate, but prefer custom policies for production.
