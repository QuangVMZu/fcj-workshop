---
title: "Monitoring, Audit, and Email Delivery"
date: 2026-04-04
weight: 6
chapter: false
pre: " <b> 4.6. </b> "
---

# Hands-on Monitoring, Audit, and Email Delivery

## Overview

- This section explains how operators observe the running platform, audit AWS activity, and verify outbound email delivery.
- It focuses on Amazon CloudWatch, AWS CloudTrail, Amazon SES, and Route 53 email DNS records.
- It is the right section when the team needs to inspect logs, trace operator actions, or validate the email path used by the application.

## Components

### Amazon CloudWatch
![Amazon CloudWatch Logs and Metrics](/images/4-Workshop/cloudwatch.png)
- Stores logs and metrics from the running environment.
- Enables monitoring, alerting, and troubleshooting of application behavior.

### AWS CloudTrail
![AWS CloudTrail Events](/images/4-Workshop/cloudtrail.png)
- Records AWS API activity across the account.
- Supports auditing, security analysis, and compliance tracking.

### Amazon SES
![Amazon SES](/images/4-Workshop/ses.png)
- Sends application emails such as notifications or verification messages.
- Requires verified domain and proper DNS configuration.

### Route 53 DNS Records (SES Support)
![Route 53 SES DNS Records](/images/4-Workshop/route53-ses.png)
- Provides DNS records for domain verification and email routing.
- Includes DKIM, SPF, and MX records for SES integration.

### IAM-backed EC2 Role
- Grants EC2 permission to:
  - Publish logs to CloudWatch
  - Send emails via SES
- Eliminates the need for hardcoded credentials.

## Why This Design Uses Direct AWS Integrations

- The diagram shows CloudWatch and CloudTrail directly, so observability is centered on AWS-native tooling.
- Email is sent directly through Amazon SES rather than via a separate messaging or event bus service.
- EventBridge, SQS, and SNS are not shown in the architecture, so this workshop does not assume an event-driven notification pipeline.

## Internal Logic and Operational Model

- CloudWatch explains what the workload is doing at runtime by storing logs, metrics, and operational signals.
- CloudTrail explains what changed in the AWS environment by capturing control-plane activity.
- SES provides managed outbound communication so the application does not have to operate its own SMTP stack.
- Route 53 supports the email trust model by hosting the DNS records that prove domain ownership and improve deliverability.
- Because these services are cross-cutting, problems in this section often explain symptoms seen in other layers even when the user-facing request path looks healthy.

## Step-by-Step AWS Flow

1. The application emits logs and runtime metrics.
   Service: Amazon CloudWatch.
   What happens: Application and platform signals are written to log groups and CloudWatch metrics.
   Why it matters: Logs and metrics are the first place to look when user-facing errors appear.
2. Operators inspect the live logs.
   Service: Amazon CloudWatch Logs.
   What happens: The team tails or queries log streams to find failures and timing issues.
   Why it matters: Fast diagnosis depends on finding the correct log group quickly.
3. AWS control-plane activity is recorded.
   Service: AWS CloudTrail.
   What happens: Actions such as starting an SSM session or changing infrastructure are stored as audit events.
   Why it matters: Operators can tell who changed what and when.
4. The application sends email through SES.
   Service: Amazon SES.
   What happens: The EC2 workload calls SES APIs with its IAM role.
   Why it matters: Notifications, OTP, or operational messages can leave the platform without self-managed SMTP servers.
5. SES relies on DNS records in Route 53.
   Services: Amazon SES and Route 53.
   What happens: Domain verification, DKIM, and mail routing records are published in the hosted zone.
   Why it matters: Without valid DNS records, SES identity verification or deliverability will fail.

## AWS CLI Walkthrough

### 1. Tail recent application logs

```bash
aws logs tail ${LOG_GROUP} \
  --since 30m \
  --follow \
  --region ${AWS_REGION}
```

### 2. Look up recent audit events

```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=StartSession \
  --max-results 10 \
  --region ${AWS_REGION}
```

### 3. Check SES identity status

```bash
aws sesv2 get-email-identity \
  --email-identity ${SES_IDENTITY} \
  --region ${AWS_REGION}
```

### 4. Send a test email through SES

Create a file named `ses-test-email.json`:

```json
{
  "FromEmailAddress": "no-reply@example.com",
  "Destination": {
    "ToAddresses": [
      "ops@example.com"
    ]
  },
  "Content": {
    "Simple": {
      "Subject": {
        "Data": "EV platform test email"
      },
      "Body": {
        "Text": {
          "Data": "If you received this message, SES delivery is working."
        }
      }
    }
  }
}
```

Send the message:

```bash
aws sesv2 send-email \
  --cli-input-json file://ses-test-email.json \
  --region ${AWS_REGION}
```

### 5. Create a simple CPU alarm for one backend instance

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name ev-app-instance-high-cpu \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=${INSTANCE_ID} \
  --evaluation-periods 2 \
  --region ${AWS_REGION}
```

## Configuration Example

### Example Route 53 records for SES domain setup

Create a file named `ses-identity-records.json`:

```json
{
  "Comment": "SES identity verification records",
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "example.com",
        "Type": "TXT",
        "TTL": 300,
        "ResourceRecords": [
          {
            "Value": "\"amazonses:verification-token\""
          }
        ]
      }
    },
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "selector1._domainkey.example.com",
        "Type": "CNAME",
        "TTL": 300,
        "ResourceRecords": [
          {
            "Value": "selector1-example-com.dkim.amazonses.com"
          }
        ]
      }
    },
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "mail.example.com",
        "Type": "MX",
        "TTL": 300,
        "ResourceRecords": [
          {
            "Value": "10 inbound-smtp.ap-southeast-1.amazonaws.com"
          }
        ]
      }
    }
  ]
}
```

Apply the DNS records:

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id ${HOSTED_ZONE_ID} \
  --change-batch file://ses-identity-records.json
```

## Best Practices and Improvement Paths

- Use structured application logs so search, filtering, and incident review become easier over time.
- Define log retention explicitly for application, audit, and security-relevant data instead of relying on defaults.
- Treat CloudTrail as part of the governance baseline, not just a troubleshooting aid.
- Add alarm routing and ownership so every important alarm has an actual response path.
- If notification volume or retry complexity grows, services such as EventBridge, SQS, or SNS can decouple email and other side effects from the synchronous request path.

## What to Verify

- Application logs arrive in the expected CloudWatch log group.
- CloudTrail captures administrator and infrastructure actions.
- SES identities are verified before the application attempts delivery.
- Route 53 contains the TXT, CNAME, and MX records required by your SES setup.
- CloudWatch alarms exist for the signals your team cares about.

## Common Mistakes

- Sending email before verifying the SES identity.
- Looking only at application logs and ignoring CloudTrail when the issue is an operator action.
- Creating alarms without deciding who will review them.
- Expecting EventBridge or SQS-driven notification workflows when the architecture only shows direct SES integration.

## Notes / Assumptions

- The diagram does not show dashboards, dashboards-as-code, tracing, or alarm actions.
- The alarm example intentionally avoids SNS wiring because SNS is not part of the visible architecture.
- Email use cases such as OTP or notifications are inferred from the presence of SES, but the exact templates are not shown in the diagram.



