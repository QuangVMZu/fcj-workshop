---
title: "Event 2 - AWS re:Invent Recap HCMC"
date: 2026-02-04
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# AWS re:Invent Recap HCMC

## Overview

AWS re:Invent Recap HCMC is a community event that summarizes the most important announcements and innovations from AWS re:Invent — the largest global cloud conference.

The session provided insights into:
- New AWS services
- Cloud architecture trends
- Real-world best practices from industry experts

![AWS re:Invent Recap HCMC](/images/3-EventParticipated/sk.jpg)

---

## Key Themes & Learnings

### 1. Cloud is Moving Towards Abstraction

A major trend highlighted:

> Developers focus less on infrastructure, more on logic.

Examples:
- Serverless (Lambda, API Gateway)
- Managed services (RDS, DynamoDB)
- AI/ML services (Bedrock, SageMaker)

💡 Insight:
> The future of cloud is **less management, more automation**.

---

### 2. Modern Architecture Patterns

We explored common patterns used in real systems:

- **Microservices architecture**
- **Event-driven systems**
- **Serverless-first approach**

Typical flow:
Client → API Gateway → Lambda → Database (RDS/DynamoDB)


Benefits:
- Scalability
- Fault isolation
- Faster development

---

### 3. Cost Optimization is a Design Responsibility

Not just finance — engineers must design for cost.

Key strategies:
- Right-sizing resources
- Using Savings Plans / Reserved Instances
- Choosing the right service (EC2 vs Lambda vs ECS)

💡 Insight:
> A well-designed system is both **scalable AND cost-efficient**.

---

### 4. Reliability & Resilience

AWS emphasizes:

- Multi-AZ deployments
- Backup strategies
- Fault tolerance

Design principle:
> “Everything fails — design for it.”

---

### 5. AI is Becoming Native in Cloud

AI is no longer optional.

Key highlights:
- Amazon Bedrock (foundation models)
- AI integration into development workflows
- Automation using AI agents

---

## Practical Takeaways

- Always choose **managed services first**
- Design systems with:
  - Scalability
  - Fault tolerance
  - Cost efficiency
- Think in **architecture patterns**, not just services
- Combine services instead of over-engineering

---

## Personal Reflection

This event helped me move from:

> “Learning AWS services individually”

to:

> “Thinking in systems and architectures”

It also showed that:

- Real-world systems are combinations of multiple services
- Good architecture is about **trade-offs**, not perfection

---

## How I Apply This

After the event, I started:

- Designing systems using **architecture patterns**
- Considering cost and scalability from the beginning
- Combining services like:
  - EC2 + RDS + S3
  - or Serverless stack (Lambda + API Gateway)

This mindset directly influenced how I approached my project implementation.
