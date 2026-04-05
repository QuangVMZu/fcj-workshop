---
title: "Workshop"
date: 2026-04-04
weight: 4
chapter: false
pre: " <b> 4. </b> "
---

# Hands-on AWS Workshop for the EV Charging Platform

![AWS Workshop Architecture Overview](/images/2-Proposal/AWS-architecture.png)

## Overview

This chapter is a hands-on workshop guide for the AWS architecture behind the EV Charging Station Management System. It does not teach local application setup. Instead, it explains how the deployed AWS services receive traffic, run the backend, store data, protect secrets, and support operations.

## How to Use This Workshop

- Follow the sections in order from edge services to observability.
- Replace placeholder values such as `${APP_DOMAIN}` and `${DB_INSTANCE_ID}` with your own environment values.
- The CLI examples use `bash` syntax. If you use PowerShell, adapt quoting accordingly.
- Prefer a non-production account for experiments such as Route 53 changes, CloudFront invalidations, or RDS failover tests.
- Read each section in two passes: first for the flow, then for the commands and verification steps.

## Services Used in This Architecture

- **Edge and frontend delivery**: Amazon Route 53, Amazon CloudFront, AWS WAF, Amazon S3, Amazon Location Service
- **Application runtime**: Amazon VPC, Internet Gateway, NAT Gateway, Application Load Balancer, Amazon EC2, EC2 Auto Scaling
- **Data layer**: Amazon RDS in a primary and standby layout
- **Operations and security**: AWS Systems Manager, IAM instance profile, AWS Secrets Manager, AWS KMS
- **Observability and communication**: Amazon CloudWatch, AWS CloudTrail, Amazon SES, Route 53 email DNS records

## Services Not Shown in This Diagram

- **API Gateway**: Not used here because public application traffic enters through CloudFront and then the Application Load Balancer.
- **Lambda**: Not shown. The backend is modeled as long-running EC2 instances.
- **ECS / EKS**: Not shown. Container orchestration is outside the scope of this design.
- **DynamoDB**: Not shown. Persistent data is modeled as relational data in Amazon RDS.
- **EventBridge / SQS / SNS**: Not shown. The diagram emphasizes synchronous request flow and direct service integrations.
- **Cognito**: Not shown. The visible admin path uses IAM and AWS Systems Manager rather than a user pool.

## What This Chapter Covers in Practice

- It explains how a browser request enters AWS, reaches the frontend, and then transitions into backend API processing.
- It clarifies where the public trust boundary ends and where the private application and data tiers begin.
- It shows how operators administer the environment without opening SSH to the internet.
- It highlights the supporting services that make the runtime production-ready, such as secret management, audit logging, metrics, and outbound email.
- It intentionally focuses on deployed AWS behavior so the reader can reason about runtime responsibilities, failure domains, and service interactions layer by layer.

## Shared Lab Variables

```bash
export AWS_REGION=ap-southeast-1
export APP_DOMAIN=app.example.com
export HOSTED_ZONE_ID=Z123456789EXAMPLE
export STATIC_BUCKET=ev-prod-frontend
export CLOUDFRONT_DIST_ID=E123ABC456DEF
export PLACE_INDEX_NAME=ev-place-index
export ALB_NAME=ev-app-alb
export TARGET_GROUP_ARN=arn:aws:elasticloadbalancing:ap-southeast-1:123456789012:targetgroup/ev-app-tg/abc123
export ASG_NAME=ev-app-asg
export INSTANCE_ID=i-0123456789abcdef0
export APP_PORT=8080
export DB_INSTANCE_ID=ev-prod-rds
export DB_PORT=1433
export DB_SECRET_ID=ev/prod/database
export LOG_GROUP=/aws/ec2/ev-app
export SES_IDENTITY=example.com
export SES_SENDER=no-reply@example.com
export SES_RECIPIENT=ops@example.com
```

## Recommended AWS Access

- Route 53 read and change permissions for DNS validation tasks
- CloudFront and S3 permissions for frontend deployment checks
- EC2, ELBv2, Auto Scaling, and VPC read permissions for application runtime inspection
- RDS read permissions, and controlled reboot permissions only in approved environments
- SSM, Secrets Manager, IAM, KMS, CloudWatch, CloudTrail, and SES read permissions for operations work

## Content

1. [Overview](4.1-overview/)
2. [Edge Delivery and Geospatial Requests](4.2-edge-delivery-and-geospatial-requests/)
3. [Private Application Routing and Compute](4.3-private-application-routing-and-compute/)
4. [Relational Data Resilience and Network Isolation](4.4-relational-data-resilience-and-network-isolation/)
5. [Operational Access and Secret Governance](4.5-operational-access-and-secret-governance/)
6. [Monitoring, Audit, and Email Delivery](4.6-monitoring-audit-and-email-delivery/)
7. [Workshop Architecture Deep Dive](4.7-workshop-architecture-deep-dive/)

