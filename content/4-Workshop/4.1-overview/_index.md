---
title: "Overview"
date: 2026-04-04
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Workshop Overview and AWS Service Map

![AWS Workshop Architecture Overview](/images/4-Workshop/ev-achi.png)

## System Overview

- The EV Charging Station Management System is deployed as a layered AWS web platform.
- End users access a browser-based frontend through Route 53 and CloudFront.
- Static assets are stored in Amazon S3 and protected behind CloudFront Origin Access Control.
- Backend requests are routed to private EC2 instances through an Application Load Balancer.
- Business data is stored in Amazon RDS and isolated from public access.
- Operations teams use AWS-native tooling such as Systems Manager, Secrets Manager, CloudWatch, CloudTrail, and SES.

## AWS Service Map

| Architecture need | Service used in this design | Why it is used | Not used here |
| --- | --- | --- | --- |
| Public DNS | Amazon Route 53 | Routes users to the public entry points | API Gateway custom domain |
| Frontend delivery | Amazon CloudFront + Amazon S3 | Delivers static assets globally with cache | Amplify Hosting |
| Edge protection | AWS WAF | Filters unwanted web traffic | Shield configuration details are not shown |
| Map lookup | Amazon Location Service | Serves geospatial search and map features | Third-party map provider |
| API entry | Application Load Balancer | Forwards HTTP traffic to private compute | API Gateway |
| Compute | Amazon EC2 + Auto Scaling | Runs long-lived backend processes | Lambda, ECS, EKS |
| Outbound internet | NAT Gateway + Internet Gateway | Allows private instances to reach external AWS endpoints or the internet | Public EC2 instances |
| Relational storage | Amazon RDS | Stores transactional data with standby support | DynamoDB |
| Admin access | AWS Systems Manager | Avoids direct SSH exposure | Bastion host |
| Secret handling | AWS Secrets Manager + AWS KMS | Centralizes and encrypts secrets | Hard-coded credentials |
| Monitoring | Amazon CloudWatch | Captures metrics and logs | Third-party APM not shown |
| Audit | AWS CloudTrail | Records AWS API activity | EventBridge audit fan-out |
| Email | Amazon SES | Sends transactional email | SNS email delivery |

## End-to-End AWS Flow

1. A user resolves `${APP_DOMAIN}` through Amazon Route 53.
   Service: Amazon Route 53.
   Why it matters: DNS is the first control point for the public application name.
2. CloudFront receives the request and evaluates AWS WAF rules.
   Services: Amazon CloudFront and AWS WAF.
   Why it matters: The edge layer reduces latency and filters malicious requests before they hit the backend.
3. CloudFront serves cached frontend assets or fetches them from Amazon S3.
   Services: Amazon CloudFront and Amazon S3.
   Why it matters: Static delivery stays fast and the S3 bucket does not need to be public.
4. The frontend calls backend APIs through the Application Load Balancer.
   Service: Application Load Balancer.
   Why it matters: The ALB is the public HTTP entry point for dynamic requests.
5. The ALB routes traffic to healthy EC2 instances in private subnets.
   Services: ALB, Amazon EC2, EC2 Auto Scaling.
   Why it matters: The backend can scale horizontally without exposing instances directly.
6. The EC2 application reads or writes relational data in Amazon RDS.
   Service: Amazon RDS.
   Why it matters: Transactions stay in a managed relational service with standby support.
7. The application retrieves secrets, writes logs, and sends email by using its IAM role.
   Services: IAM, Secrets Manager, KMS, CloudWatch, SES.
   Why it matters: The instance does not need static credentials stored on disk.
8. Operators administer the environment through Systems Manager and audit activities in CloudTrail.
   Services: AWS Systems Manager and AWS CloudTrail.
   Why it matters: Operations remain traceable and do not require public SSH access.

## Hands-on Baseline Checks

Run these commands before working through the remaining sections.

```bash
aws sts get-caller-identity
aws route53 list-hosted-zones --region ${AWS_REGION}
aws cloudfront list-distributions --region ${AWS_REGION}
aws ec2 describe-instances --region ${AWS_REGION} --max-items 5
aws rds describe-db-instances --region ${AWS_REGION}
```

What to look for:

- You can authenticate to the intended AWS account.
- The hosted zone, CloudFront distribution, EC2 instances, and RDS instance all exist in the expected environment.
- The region and naming convention match the values you plan to use in the later labs.

## Beginner Guidance

- Start by understanding which components are public and which are private.
- Treat Route 53, CloudFront, ALB, and SES as public-facing services.
- Treat EC2, RDS, Secrets Manager, and KMS as controlled internal services.
- Treat Systems Manager, CloudWatch, and CloudTrail as operator tools.
- If a service is not shown in the diagram, do not assume it exists in the deployment.

## Notes / Assumptions

- The diagram shows infrastructure topology, not application source code or CI/CD pipelines.
- The existing project proposal mentions Amazon RDS for SQL Server, but the diagram itself labels only Amazon RDS. Adjust engine-specific commands as needed.
- No API Gateway, Lambda, ECS, EKS, DynamoDB, EventBridge, SQS, SNS, or Cognito components are visible in this architecture.


