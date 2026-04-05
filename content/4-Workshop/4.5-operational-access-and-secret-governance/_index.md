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
- It is the right section to review when the team needs to reason about how people and workloads gain trust without relying on public SSH or hardcoded credentials.

## Components

### AWS Systems Manager (SSM)
![AWS Systems Manager Session Manager](/images/4-Workshop/ssm-session-manager.png)
- Provides controlled access for operators to connect to private EC2 instances.
- Eliminates the need for SSH keys or public access.
- Can support interactive sessions, command execution, and standardized operational workflows.

### IAM Instance Profile
![IAM Instance Profile](/images/4-Workshop/iam-instance-profile.png)
- Grants machine-level identity to EC2 instances.
- Allows applications to securely access AWS services without hardcoded credentials.
- Supplies temporary credentials that AWS rotates automatically.

### AWS Secrets Manager
- Stores sensitive data such as database connection strings, API keys, and credentials.
- Provides secure retrieval of secrets at runtime.
- Separates secret lifecycle from application deployment lifecycle.

### AWS KMS
- Encrypts and protects secrets managed by the system.
- Controls access to encryption keys via IAM policies.
- Makes encryption governance a managed platform capability rather than a custom application concern.

### Amazon EC2
![EC2 Instance IAM Role](/images/4-Workshop/ec2-iam-role.png)
- Consumes IAM roles and retrieves secrets during application runtime.
- Acts as the execution environment for secure service interaction.
- Uses machine identity to reach Secrets Manager, CloudWatch, SES, and other AWS APIs.

## Why This Design Uses SSM and IAM

- The diagram shows operator access flowing through Systems Manager instead of a direct public login path.
- EC2 instances use an attached IAM role, so the application does not need long-lived AWS keys stored on disk.
- The visible access path is an operator and machine identity path, not an end-user identity path.
- This design keeps access auditable, revocable, and aligned with AWS-native trust models.

## Flow Description

1. An administrator reaches the environment through AWS Systems Manager rather than a direct internet-facing login path.
   What happens: The operator starts a brokered session to a private instance through the SSM control plane.
   Why it matters: The instance remains private while still being operable.
2. EC2 instances assume their IAM instance profile automatically at runtime.
   What happens: Temporary AWS credentials are issued through the instance metadata service.
   Why it matters: The workload can call AWS APIs without storing static keys.
3. The application reads secrets from AWS Secrets Manager when configuration or credentials are required.
   What happens: Secret values are fetched by name during startup or runtime.
   Why it matters: Secret rotation and environment separation become easier to manage.
4. AWS KMS likely protects the secret material or encryption operations associated with those secrets.
   What happens: Authorized callers can decrypt protected values, while unauthorized callers are denied.
   Why it matters: Encryption and decryption stay under managed policy control.
5. The same IAM role also allows the instance to write logs and send email, as shown in the diagram.
   What happens: One machine identity supports multiple supporting integrations while still remaining auditable.
   Why it matters: The platform avoids credential sprawl across the application estate.

## Request and Response Behavior

- An operator request reaches SSM, which either establishes a managed session or fails due to registration or IAM issues.
- An application request reaches Secrets Manager, which either returns the secret value or fails due to permission, region, or naming errors.
- A decryption request reaches KMS implicitly through the secret flow and succeeds only when IAM policy and key policy align.
- The runtime either receives the configuration it needs to continue safely or surfaces a policy/configuration failure early.

## AWS CLI Walkthrough

### 1. Start an SSM session to a private instance

```bash
aws ssm start-session \
  --target ${INSTANCE_ID} \
  --region ${AWS_REGION}
```

### 2. Verify the attached IAM instance profile

```bash
aws ec2 describe-instances \
  --instance-ids ${INSTANCE_ID} \
  --region ${AWS_REGION} \
  --query 'Reservations[0].Instances[0].IamInstanceProfile.Arn' \
  --output text
```

### 3. Read a secret value

```bash
aws secretsmanager get-secret-value \
  --secret-id ${DB_SECRET_ID} \
  --region ${AWS_REGION}
```

### 4. List KMS aliases

```bash
aws kms list-aliases --region ${AWS_REGION}
```

How to interpret these commands:

- A successful SSM session proves both management-plane reachability and correct instance registration.
- A missing instance profile ARN often explains later failures in secret retrieval, log publishing, or SES usage.
- Secret retrieval failures usually point to IAM permissions, incorrect secret names, or missing KMS access.
- KMS alias discovery helps verify whether the expected encryption boundary exists in the environment.

## Configuration Example

### Minimum permissions for the instance role in this architecture

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue",
        "kms:Decrypt",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "ses:SendEmail",
        "ses:SendRawEmail"
      ],
      "Resource": "*"
    }
  ]
}
```

What this policy achieves:

- The instance can read secrets.
- The instance can decrypt protected data through KMS.
- The instance can publish logs and send email.
- The workload does not require static AWS keys stored on the server.

## Key Logic / Rules

- Administrative access is brokered through SSM, which avoids the need for public SSH or bastion exposure in the diagram.
- Secret values are externalized into a managed service instead of being embedded directly on application servers.
- IAM roles enforce least-privilege access for machine identities.
- Encryption is treated as a managed capability rather than an application-owned key store.
- The instance profile shown in the diagram is scoped to reading secrets, writing logs, and sending email.
- In production, resource scope should be narrower than the simplified workshop examples.

## Operational and Security Considerations

- Separate operator permissions from workload permissions so human access and machine access remain distinct.
- Use Session Manager logging for traceability when interactive access is allowed.
- Use consistent secret naming by environment, workload, and purpose.
- Review KMS key policy scope carefully when customer-managed keys are used.
- Consider VPC endpoints for SSM, Secrets Manager, and CloudWatch if the environment needs to reduce dependence on NAT for control-plane access.

## Best Practices

- Replace wildcard resource access with secret-specific and key-specific ARNs in production.
- Document which workloads may decrypt which secrets and why.
- Rotate secrets on a schedule that matches the environment's risk profile.
- Treat failed secret retrieval as an operational signal, not just an application bug.

## What to Verify

- The target EC2 instance is registered in Systems Manager.
- The instance has an IAM instance profile attached.
- The application can read the exact secret names it requires.
- KMS permissions are granted only to the principals that truly need decryption.
- Operators use SSM instead of opening inbound SSH from the internet.

## Common Mistakes

- Attaching a role to EC2 but forgetting `secretsmanager:GetSecretValue`.
- Granting Secrets Manager access but missing `kms:Decrypt` when a customer-managed key is used.
- Assuming end-user identity is handled here when the diagram only shows operator and machine access.
- Returning to hardcoded credentials because secret naming is inconsistent.
- Leaving workshop-style broad IAM permissions in place after the environment matures.

## Notes / Assumptions

- The exact SSM features in use, such as Session Manager or Run Command, are not labeled explicitly.
- Secret rotation and KMS key rotation policies are not shown.
- No separate identity provider or fine-grained admin workflow engine appears in the diagram, so the documentation stays focused on the visible AWS controls.
- In a hardened environment, private VPC endpoints for SSM, Secrets Manager, and CloudWatch may reduce dependence on NAT for control-plane traffic.