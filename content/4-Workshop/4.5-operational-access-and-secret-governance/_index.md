---
title: "Operational Access and Secret Governance"
date: 2026-04-04
weight: 5
chapter: false
pre: " <b> 4.5. </b> "
---

# Operational Access and Secret Governance

## Overview

- This section explains how administrators and application instances receive controlled access to the environment and sensitive configuration.
- Its purpose is to reduce direct exposure, centralize secret handling, and enforce machine permissions through AWS-native controls.
- It fits as the security and operations control plane alongside the runtime environment.

## Components

### AWS Systems Manager (SSM)
![AWS Systems Manager Session Manager](/images/4-Workshop/ssm-session-manager.png)
- Provides controlled access for operators to connect to private EC2 instances.
- Eliminates the need for SSH keys or public access.

### IAM Instance Profile
![IAM Instance Profile](/images/4-Workshop/iam-instance-profile.png)
- Grants machine-level identity to EC2 instances.
- Allows applications to securely access AWS services without hardcoded credentials.

### AWS Secrets Manager
- Stores sensitive data such as database connection strings, API keys, and credentials.
- Provides secure retrieval of secrets at runtime.

### AWS KMS
- Encrypts and protects secrets managed by the system.
- Controls access to encryption keys via IAM policies.

### Amazon EC2
![EC2 Instance IAM Role](/images/4-Workshop/ec2-iam-role.png)
- Consumes IAM roles and retrieves secrets during application runtime.
- Acts as the execution environment for secure service interaction.

## Flow Description

1. An administrator reaches the environment through AWS Systems Manager rather than a direct internet-facing login path.
2. EC2 instances assume their IAM instance profile automatically at runtime.
3. The application reads secrets from AWS Secrets Manager when configuration or credentials are required.
4. AWS KMS likely protects the secret material or encryption operations associated with those secrets.
5. The same IAM role also allows the instance to write logs and send email, as shown in the diagram.

## Key Logic / Rules

- Administrative access is brokered through SSM, which avoids the need for public SSH or bastion exposure in the diagram.
- Secret values are externalized into a managed service instead of being embedded directly on application servers.
- IAM roles enforce least-privilege access for machine identities.
- Encryption is treated as a managed capability rather than an application-owned key store.
- The instance profile shown in the diagram is scoped to reading secrets, writing logs, and sending email.

## Notes / Assumptions

- The exact SSM features in use, such as Session Manager or Run Command, are not labeled explicitly.
- Secret rotation and KMS key rotation policies are not shown.
- No separate identity provider or fine-grained admin workflow engine appears in the diagram, so the documentation stays focused on the visible AWS controls.
