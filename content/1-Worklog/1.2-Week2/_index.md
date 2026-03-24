---
title: "Week 2 Worklog"
date: 2026-01-12
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives

This week focused on building a strong foundation in AWS identity management and networking, progressing from basic to advanced concepts. The key objectives included:

- **IAM Management**:
  - Create Users and Groups
  - Attach appropriate Policies
  - Create and use IAM Roles
  - Practice **Switch Role**

- **Understand VPC Networking**:
  - Overview of VPC architecture
  - Compare **Security Groups vs Network ACLs**
  - Prepare networking environment for EC2

- **Deploy EC2**:
  - Launch EC2 instances in subnets
  - Configure Security Groups
  - Connect via SSH using Key Pair

- **Configure Hybrid DNS with Route 53 Resolver**:
  - Create Key Pair
  - Configure Security Groups
  - Configure DNS:
    - Create Outbound Endpoint
    - Create Resolver Rules
    - Create Inbound Endpoint
  - Clean up resources

- **Set up VPC Peering**:
  - Introduction to Peering
  - Use CloudFormation for infrastructure setup
  - Create Security Groups & EC2 instances
  - Update Network ACLs
  - Establish Peering Connection
  - Update Route Tables
  - Enable Cross-Peer DNS

- **Deploy AWS Transit Gateway**:
  - Create Transit Gateway
  - Create Attachments for each VPC
  - Configure Transit Gateway Route Tables
  - Update VPC Route Tables

---

### Tasks Overview

| Day | Task | Start Date | Completion Date | References |
| :-: |------|:----------:|:---------------:|------------|
|  2  | **IAM & EC2 Basics**:<br>- Create Users & Groups<br>- Attach Policies<br>- Create Role & Switch Role<br>- Launch EC2 + SSH | 12/01/2026 | 12/01/2026 | [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj/) |
|  3  | **VPC Networking**:<br>- VPC Overview<br>- Security Group vs NACL<br>- Subnet preparation | 13/01/2026 | 13/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  4  | **Hybrid DNS (Route 53 Resolver)**:<br>- Key Pair<br>- Security Groups<br>- Outbound / Inbound Endpoints<br>- Resolver Rules | 14/01/2026 | 14/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  5  | **VPC Peering**:<br>- CloudFormation Template<br>- EC2 & SG setup<br>- Peering connection<br>- Route Tables update<br>- Enable DNS | 15/01/2026 | 15/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |
|  6  | **Transit Gateway**:<br>- Create TGW<br>- Attach VPCs<br>- Configure routing | 16/01/2026 | 16/01/2026 | [AWS Docs](https://cloudjourney.awsstudygroup.com/) |

---

### Week 2 Achievements

#### What was accomplished

- Gained a solid understanding of **IAM** and successfully practiced **Switch Role**
- Mastered **VPC networking** and security mechanisms using Security Groups
- Successfully deployed and connected to an **EC2 instance via SSH**
- Configured **Hybrid DNS** using Route 53 Resolver
- Implemented **VPC Peering** between Dev and Staging environments
- Deployed **AWS Transit Gateway** as a central networking hub
- Practiced **Infrastructure as Code (IaC)** using CloudFormation
- Completed proper **resource cleanup**

#### Architecture Summary

- **IAM Structure**: Users → Groups → Policies → Roles for flexible access control
- **VPC Security Model**:
  - Security Group: instance-level firewall
  - NACL: subnet-level firewall
- **Hybrid DNS**:
  - On-premises ↔ AWS via Route 53 Resolver Endpoints
- **Networking**:
  - VPC Peering: point-to-point connection
  - Transit Gateway: hub-and-spoke architecture
