---
title: "Private Application Routing and Compute"
date: 2026-04-04
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Hands-on Private Application Routing and Compute

## Overview

- This section explains how dynamic requests move from the edge into private compute.
- It focuses on the VPC, public and private subnets, Internet Gateway, NAT Gateway, ALB, EC2 instances, and Auto Scaling.
- It is the primary section for troubleshooting `5xx` errors, unhealthy targets, or outbound connectivity issues.

## AWS Services in This Section

### Amazon VPC
![Amazon VPC](/images/4-Workshop/vpc-overview.png)
- Defines the network boundary for the entire system.
- Hosts all subnets, gateways, and compute resources.

### Internet Gateway
![Internet Gateway](/images/4-Workshop/internet-gateway.png)
- Connects the VPC to the public internet.
- Enables inbound and outbound traffic based on route table configuration.

### NAT Gateway
![NAT Gateway](/images/4-Workshop/nat-gateway.png)
- Allows instances in private subnets to access the internet.
- Does not allow direct inbound traffic from the internet.

### Application Load Balancer
![Application Load Balancer](/images/4-Workshop/alb.png)
- Receives external HTTP/HTTPS requests.
- Distributes traffic to backend EC2 instances.

### Amazon EC2
![Amazon EC2 Instances](/images/4-Workshop/ec2-instances.png)
- Runs backend application workloads.
- Processes business logic and returns responses.

### EC2 Auto Scaling
![Auto Scaling Group](/images/4-Workshop/auto-scaling-group.png)
- Automatically scales the number of instances based on load.
- Ensures high availability across multiple Availability Zones.

## Why This Design Uses ALB + EC2

- The diagram shows long-running application servers, so **Amazon EC2** is the active compute model.
- The public API entry point is **Application Load Balancer**, not API Gateway.
- No Lambda, ECS, or EKS components are shown, so this workshop does not assume serverless or container orchestration.
- Private subnets indicate that compute should not be directly reachable from the internet.

## Step-by-Step AWS Flow

1. The frontend sends a dynamic request to the backend hostname.
   Service: Application Load Balancer.
   What happens: The ALB receives the request at the public boundary of the application tier.
   Why it matters: Only the ALB should be public for backend traffic.
2. The ALB evaluates listener rules and target group health.
   Service: Application Load Balancer.
   What happens: Requests are routed only to registered healthy targets.
   Why it matters: Bad instances can be isolated without taking down the whole service.
3. A healthy EC2 instance in a private subnet processes the request.
   Service: Amazon EC2.
   What happens: The application performs business logic and prepares the response.
   Why it matters: Compute remains private and controlled by security groups.
4. Auto Scaling keeps the desired number of instances running.
   Service: EC2 Auto Scaling.
   What happens: Failed instances can be replaced and capacity can grow during traffic spikes.
   Why it matters: The platform can recover from instance failure and scale out.
5. If the instance needs outbound access, it sends traffic through the NAT Gateway.
   Services: NAT Gateway and Internet Gateway.
   What happens: Private instances reach external endpoints without receiving public IPs.
   Why it matters: The backend can call AWS public service endpoints or download dependencies without becoming internet-facing.
6. The response returns through the ALB to the client.
   Service: Application Load Balancer.
   What happens: The ALB completes the HTTP request lifecycle.
   Why it matters: Clients never communicate with EC2 instances directly.

## AWS CLI Walkthrough

### 1. Inspect the load balancer

```bash
aws elbv2 describe-load-balancers \
  --names ${ALB_NAME} \
  --region ${AWS_REGION}
```

### 2. Check target health

```bash
aws elbv2 describe-target-health \
  --target-group-arn ${TARGET_GROUP_ARN} \
  --region ${AWS_REGION}
```

### 3. Inspect the Auto Scaling group

```bash
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names ${ASG_NAME} \
  --region ${AWS_REGION}
```

### 4. Inspect a backend instance

```bash
aws ec2 describe-instances \
  --instance-ids ${INSTANCE_ID} \
  --region ${AWS_REGION}
```

## Configuration Example

### Typical traffic rules for this architecture

| Component | Inbound rule | Outbound rule | Purpose |
| --- | --- | --- | --- |
| ALB security group | `443` from `0.0.0.0/0` | `${APP_PORT}` to EC2 security group | Public HTTPS entry |
| EC2 security group | `${APP_PORT}` from ALB security group | Database port to RDS, `443` for outbound AWS calls | Private application runtime |
| Private subnet route table | No direct internet route | `0.0.0.0/0` to NAT Gateway | Controlled egress |
| Public subnet route table | `0.0.0.0/0` to Internet Gateway | Public edge path | Supports ALB and NAT Gateway |

### Recommended ALB health check settings

- Protocol: `HTTP` or `HTTPS`
- Health check path: a lightweight endpoint such as `/actuator/health` or `/health`
- Success codes: `200-399`
- Interval: `30` seconds
- Healthy threshold: `2` or more
- Unhealthy threshold: `2` or more

## What to Verify

- The ALB is internet-facing, but EC2 instances are in private subnets.
- Target health is `healthy` for every active instance.
- The Auto Scaling group spans at least two Availability Zones.
- Private subnets use a NAT route for outbound traffic.
- Backend responses work even though instances do not have public IPs.

## Common Mistakes

- Opening the EC2 security group to the internet instead of only to the ALB.
- Using a health check path that depends on the database and marks healthy instances as failed during database events.
- Forgetting that private instances still need outbound connectivity through NAT.
- Assuming API Gateway or Lambda is involved when the real entry point is the ALB.

## Notes / Assumptions

- The diagram shows one NAT Gateway. In production, you may want one per Availability Zone for stronger outbound resilience.
- Security groups and NACLs are not drawn, but they are critical to making this routing model work.
- This workshop documents EC2-based deployment because ECS and EKS are not shown in the architecture.


