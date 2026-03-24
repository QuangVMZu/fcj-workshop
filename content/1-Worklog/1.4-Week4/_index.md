---
title: "Week 4 Worklog"
date: 2026-01-26
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives

This week focused on operating systems in a production-ready environment, covering automation, security, reliability, performance, and cost optimization based on the AWS Well-Architected Framework.

- **Operational Excellence**:
  - Automate tasks using AWS Lambda (EC2 shutdown, Slack notifications)
  - Build monitoring systems with CloudWatch and Grafana
  - Manage EC2 resources using Tags
  - Automate operations with AWS Systems Manager

- **Security**:
  - Apply IAM Permission Boundaries
  - Audit security using AWS Security Hub
  - Protect applications with AWS WAF

- **Reliability**:
  - Implement backup strategies with AWS Backup
  - Connect VPCs via VPC Peering
  - Centralize networking using Transit Gateway

- **Performance Efficiency**:
  - Containerize applications with Docker and deploy on ECS
  - Build CI/CD pipelines using CodePipeline
  - Use File Storage Gateway for scalable storage

- **Cost Optimization**:
  - Apply Savings Plans and Reserved Instances
  - Perform EC2 right-sizing
  - Visualize and monitor cost usage

---

### Tasks Overview

| Day | Task | Start Date | Completion Date | References |
| :-: |------|:----------:|:---------------:|------------|
|  2  | **Automation & Monitoring**:<br>- AWS Lambda<br>- CloudWatch + Grafana<br>- EC2 Tagging<br>- Systems Manager | 26/01/2026 | 26/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  3  | **Security**:<br>- IAM Permission Boundary<br>- Security Hub<br>- AWS WAF | 27/01/2026 | 27/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  4  | **Reliability**:<br>- AWS Backup<br>- VPC Peering<br>- Transit Gateway | 28/01/2026 | 28/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  5  | **Performance**:<br>- Docker + ECS<br>- CodePipeline<br>- File Storage Gateway | 29/01/2026 | 29/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  6  | **Cost Optimization**:<br>- Savings Plans<br>- Reserved Instances<br>- Cost Visualization | 30/01/2026 | 30/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |

---

### Week 4 Achievements

#### What was accomplished

- Automated operations:
  - Automated EC2 shutdown and notifications using Lambda
  - Built monitoring dashboards with CloudWatch and Grafana
  - Managed resources using Tags and Systems Manager

- Enhanced security:
  - Implemented **IAM Permission Boundaries**
  - Deployed **AWS WAF**
  - Audited security with **Security Hub**

- Improved reliability:
  - Implemented automated backups with **AWS Backup**
  - Ensured stable networking via **VPC Peering** and **Transit Gateway**

- Optimized performance:
  - Deployed containerized applications using **Docker + ECS**
  - Built CI/CD pipelines

- Optimized cost:
  - Applied **Savings Plans / Reserved Instances**
  - Performed **EC2 right-sizing**
  - Monitored and visualized cost usage

#### Architecture Summary

- **Automation**: Lambda + Systems Manager
- **Monitoring**: CloudWatch + Grafana
- **Security**: IAM Boundary + WAF + Security Hub
- **Networking**: VPC Peering + Transit Gateway
- **Compute**: ECS (Docker)
- **CI/CD**: CodePipeline
- **Storage**: File Storage Gateway
- **Cost**: Savings Plans + Cost Monitoring
