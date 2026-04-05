---
title: "Workshop Architecture Deep Dive"
date: 2026-04-05
weight: 7
chapter: false
pre: " <b> 4.7. </b> "
---

# Production-Grade Architecture Deep Dive for the AWS Workshop

![AWS Workshop Architecture Overview](/images/2-Proposal/AWS-architecture.png)

## Overview

- This document synthesizes the existing source files under `content/4-Workshop/` and turns the workshop chapter into a single production-grade architecture reference.
- The workshop source is documentation-first rather than application-code-first, so the "modules" identified below are architecture capability modules inferred from sections `4.1` through `4.6`.
- The current runtime pattern is clear and consistent across the chapter: Route 53 and CloudFront form the public edge, Amazon S3 hosts static assets, the backend runs on private EC2 behind an Application Load Balancer, Amazon RDS is the relational system of record, and AWS-native services handle operations, secrets, monitoring, audit, and email.
- Where the original workshop leaves implementation details implicit, this document fills them in with clearly marked production-grade assumptions so the architecture reads as a complete AWS service design.

## How to Use This Deep Dive

- Read this section after `4.1` through `4.6` so the architectural narrative matches the hands-on order of the workshop.
- Use it during design reviews, onboarding, troubleshooting, or release-readiness checks when the team needs one consolidated architecture view.
- Treat services drawn in the existing diagrams as confirmed parts of the current design, and treat the added controls described here as supporting components reasonably inferred from those diagrams.
- When future-fit services such as API Gateway, Lambda, Step Functions, EventBridge, SQS, or SNS are mentioned, they are improvement paths rather than claims about the current deployed stack.

## Source Scope

| Workshop sources reviewed | Responsibility in the chapter | Architectural signals extracted |
| --- | --- | --- |
| `4-Workshop/_index.md` and `4-Workshop/_index.vi.md` | Define chapter scope, shared variables, recommended access, and services in or out of scope | Establish the canonical workshop boundary and confirm that ALB, EC2, RDS, SSM, Secrets Manager, KMS, CloudWatch, CloudTrail, and SES are the primary AWS building blocks |
| `4-Workshop/4.1-overview/_index.md` and `4-Workshop/4.1-overview/_index.vi.md` | Introduce the service map and the end-to-end request flow | Confirm the layered architecture and the public/private boundary of the platform |
| `4-Workshop/4.2-edge-delivery-and-geospatial-requests/_index.md` and `4-Workshop/4.2-edge-delivery-and-geospatial-requests/_index.vi.md` | Explain DNS, CDN delivery, WAF filtering, S3 origin protection, and Amazon Location Service usage | Define the edge module, static content path, and geospatial integration path |
| `4-Workshop/4.3-private-application-routing-and-compute/_index.md` and `4-Workshop/4.3-private-application-routing-and-compute/_index.vi.md` | Describe VPC networking, ALB routing, EC2 runtime, private subnet placement, and Auto Scaling | Define the synchronous API execution path and the network isolation model for compute |
| `4-Workshop/4.4-relational-data-resilience-and-network-isolation/_index.md` and `4-Workshop/4.4-relational-data-resilience-and-network-isolation/_index.vi.md` | Describe Amazon RDS, primary and standby behavior, and non-public database access | Define the transactional persistence model and data-tier isolation rules |
| `4-Workshop/4.5-operational-access-and-secret-governance/_index.md` and `4-Workshop/4.5-operational-access-and-secret-governance/_index.vi.md` | Describe operator access, IAM machine identity, secret retrieval, and KMS protection | Define the security control plane and secret consumption model |
| `4-Workshop/4.6-monitoring-audit-and-email-delivery/_index.md` and `4-Workshop/4.6-monitoring-audit-and-email-delivery/_index.vi.md` | Describe logging, audit, SES-based mail delivery, and supporting DNS records | Define the observability plane, audit trail, and outbound communication model |

## Reading the Workshop as an Architecture Codebase

- The workshop chapter is structured like a layered infrastructure module system.
- Each subsection introduces one platform responsibility, its AWS services, its operational commands, and its verification expectations.
- The chapter index acts like a root architecture manifest because it declares shared variables, access requirements, and intentionally excluded services.
- The subsection pages act like implementation modules because they describe behavior, provide CLI-based inspection or change commands, and define success criteria.
- The English and Vietnamese files are equivalent content variants rather than separate technical designs, so both language trees reinforce the same architecture.

## Architecture Baseline

The workshop describes the following AWS runtime baseline:

```text
User -> Route 53 -> CloudFront/WAF -> S3
Frontend -> Amazon Location Service
Frontend -> ALB -> Private EC2 -> Amazon RDS
Private EC2 -> Secrets Manager/KMS -> CloudWatch -> SES
Admin -> Systems Manager -> Private EC2
AWS control-plane activity -> CloudTrail
```

### Confirmed Design Position

- Public traffic is terminated at the edge and the application entry point, not on the EC2 instances.
- Static and dynamic traffic are deliberately separated.
- Backend compute is long-running and instance-based, so EC2 is the active execution model.
- Persistent business data is relational and private, so Amazon RDS is the active data model.
- Operator access is AWS-native and avoids direct public SSH exposure.
- The visible architecture is primarily synchronous. No queue, event bus, or state-machine orchestration layer is shown in the current workshop sources.

### Inferred but Required Building Blocks

The workshop does not draw every low-level AWS component, but the design depends on the following elements to function correctly:

| Inferred building block | Why it is required |
| --- | --- |
| AWS Certificate Manager certificates | HTTPS on CloudFront and the ALB requires valid TLS certificates even though the diagrams do not label ACM explicitly |
| Security groups | The private/public segmentation described in the workshop depends on security group rules between CloudFront, ALB, EC2, and RDS |
| Route tables and subnet associations | Internet Gateway and NAT Gateway behavior only works when public and private subnet route tables are configured correctly |
| Launch template or launch configuration | The Auto Scaling Group must know which AMI, instance type, security groups, and user data to apply to new EC2 instances |
| RDS subnet group | A private relational data tier across multiple subnets requires an explicit DB subnet group |
| SSM Agent and CloudWatch log shipping | Private instance management and log delivery normally rely on an installed and functioning SSM agent and logging integration |
| IAM trust policies and KMS key policies | Instance profiles, secret decryption, and runtime AWS API access all depend on correct trust and resource policies |

## Core Modules, Services, APIs, and Data Flow

| Core module | AWS services in scope | Main inputs | Main outputs | Immediate dependencies |
| --- | --- | --- | --- | --- |
| Edge delivery module | Route 53, CloudFront, AWS WAF, Amazon S3 | DNS lookups, HTTPS static asset requests | HTML, JavaScript, CSS, cached responses | Hosted zone, CloudFront distribution, S3 bucket, OAC, TLS certificate |
| Geospatial integration module | Amazon Location Service | Search text, coordinates, map tile requests | Places, map metadata, geospatial responses | Frontend configuration, IAM or signed access pattern, regional service availability |
| API ingress module | Application Load Balancer | HTTPS API requests from the frontend | Routed requests to healthy backend targets | Listener rules, target groups, health checks, public subnets |
| Application runtime module | VPC, private subnets, EC2, Auto Scaling, NAT Gateway, Internet Gateway | Routed API requests, runtime configuration, outbound dependency calls | Business responses, logs, emails, database queries | AMI, instance profile, security groups, route tables, NAT egress |
| Relational persistence module | Amazon RDS primary and standby layout | SQL reads, writes, transactions, reconnect events | Durable records, failover continuity, private endpoint access | DB subnet group, security group rules, engine configuration, backups |
| Security and operations control module | Systems Manager, IAM instance profile, Secrets Manager, KMS | Admin session requests, secret retrieval requests, AWS API auth flows | Temporary credentials, decrypted secret values, controlled shell access | SSM registration, IAM trust, KMS key permissions, secret naming |
| Observability and communication module | CloudWatch, CloudTrail, SES, Route 53 email DNS records | Logs, metrics, AWS API events, email send requests | Troubleshooting signals, audit history, outbound email delivery | IAM permissions, verified SES identity, DNS verification records |

## Interface and API Inventory

| Interface | Producer | Consumer | Protocol or API style | Purpose |
| --- | --- | --- | --- | --- |
| Public domain resolution | Browser | Route 53 | DNS | Resolve the workshop application domain to the CloudFront distribution |
| Static frontend delivery | Browser | CloudFront and S3 | HTTPS GET | Deliver SPA assets with caching and private-origin enforcement |
| Edge protection | CloudFront | AWS WAF | Web ACL inspection | Filter malformed or malicious requests before origin access |
| Geospatial search | Frontend | Amazon Location Service | HTTPS JSON APIs | Search charging locations and map-related data |
| Dynamic application API | Frontend | ALB to EC2 | HTTPS JSON REST-style traffic | Execute authentication, booking, charging, payment, and operator workflows |
| Database session | EC2 application | Amazon RDS | SQL over engine-specific port | Persist and query transactional business data |
| Secret retrieval | EC2 application | Secrets Manager | AWS control-plane API | Load database credentials and other runtime configuration values |
| Secret decryption | Secrets Manager | AWS KMS | Managed key usage | Protect secret values at rest and during authorized retrieval |
| Operator access | Administrator | Systems Manager | Session Manager control channel | Reach private instances without bastion hosts or public SSH |
| Log and metric publication | EC2 application or agent | CloudWatch | Log and metric ingestion APIs | Preserve runtime observability data |
| Audit recording | AWS services | CloudTrail | Service-generated audit events | Record administrative and control-plane activity |
| Email sending | EC2 application | Amazon SES | `SendEmail` or `SendRawEmail` style API calls | Deliver notification or transactional email |

## AWS Service Mapping and Architecture Expansion

| Capability | Current workshop mapping | Why this mapping fits the visible design | Production-grade expansion path |
| --- | --- | --- | --- |
| Public DNS | Route 53 | The workshop uses a named public domain and DNS is the first hop in every flow | Add health-check routing, weighted cutovers, or disaster recovery records if multi-region is introduced |
| CDN and static web delivery | CloudFront plus S3 | The frontend is static and benefits from global caching with a private origin | Add stricter cache policies, response headers policies, and signed URLs if premium content is ever introduced |
| Edge security | AWS WAF | The workshop explicitly positions request filtering at the edge | Add managed rule groups, IP reputation lists, rate-based rules, and bot controls |
| Public API exposure | Application Load Balancer | The backend is EC2-based and the workshop clearly states that API Gateway is not currently used | If the product later needs public API monetization, authorizers, request validation, or per-client throttling, API Gateway can sit in front of ALB or front separate service endpoints |
| Compute runtime | EC2 plus Auto Scaling | Long-running backend processes and health-check-based replacement align with instance-based compute | If future jobs become short-lived or highly event-driven, Lambda can be introduced for utility functions without replacing the main EC2 runtime |
| Object storage | Amazon S3 | Static frontend assets fit object storage with CloudFront OAC protection | Add S3 versioning, lifecycle transitions, and deployment prefixes for rollback support |
| Relational data | Amazon RDS | The workshop models transactional data and a primary/standby posture, which fits RDS | If read scale or cross-service isolation grows, add read replicas, Aurora migration planning, or a reporting database |
| Key and secret management | Secrets Manager plus KMS | The workshop intentionally externalizes credentials and encrypts them | Add automatic rotation, tighter resource scoping, and per-environment KMS keys |
| Private operator access | Systems Manager plus IAM | This matches the no-public-SSH requirement in the workshop | Add Session Manager logging, just-in-time access approval, and document-based operational runbooks |
| Monitoring and audit | CloudWatch plus CloudTrail | The workshop keeps observability and control-plane audit AWS-native | Add alarms, dashboards, log retention policies, metric filters, and centralized security analytics |
| Email delivery | Amazon SES plus Route 53 DNS | The application needs transactional email and SES fits the direct integration model shown | Add bounce and complaint processing, dedicated sending domains, and suppression handling |
| Workflow orchestration | Not present in the current workshop | The visible request flow is synchronous and does not require state-machine orchestration yet | Step Functions is a natural fit if future charging-session settlement or compliance workflows become multi-step and long-running |
| Async eventing | Not present in the current workshop | No EventBridge, SQS, or SNS path is shown in the diagrams or section text | EventBridge, SQS, and SNS are strong next additions for notification fan-out, retry isolation, and post-transaction background processing |
| NoSQL persistence | Not present in the current workshop | The current system of record is relational, so DynamoDB is intentionally out of scope | DynamoDB becomes relevant only if the platform later needs ultra-low-latency key-value access, idempotency stores, or event-driven denormalized views |

## Detailed Workshop Breakdown

### 4-Workshop Root Chapter

What this part does:

- Establishes the shared vocabulary and service boundary for the rest of the chapter.
- Declares common lab variables such as `${APP_DOMAIN}`, `${STATIC_BUCKET}`, `${ALB_NAME}`, `${DB_INSTANCE_ID}`, and `${SES_IDENTITY}`.
- Explains which services are intentionally not part of the visible design so readers do not invent components that the workshop never intended to cover.

Why it exists:

- It acts as the architecture manifest for the workshop.
- It keeps all later hands-on pages aligned on the same AWS region, identifiers, and operational expectations.
- It prevents scope drift by stating that API Gateway, Lambda, ECS, EKS, DynamoDB, EventBridge, SQS, SNS, and Cognito are not part of the current visible implementation.

Input and output behavior:

| Input or trigger | Processing in this root chapter | Output or effect |
| --- | --- | --- |
| Reader enters the workshop chapter | Shared context is established | The reader knows which AWS services are in scope and how the chapter is organized |
| Reader needs CLI context | Environment variable list is provided | Commands in later sections can be reused consistently |
| Reader asks what is not implemented | Excluded-service list is stated explicitly | The architecture stays anchored to the visible design rather than speculative alternatives |

High-level explanation of the workshop artifacts:

- The shared environment variables are not application code. They are an operator convenience layer that makes every later CLI command reproducible.
- The recommended AWS access list functions like an IAM readiness checklist for workshop participants.
- The content list defines the architecture decomposition order: edge first, runtime second, data third, operations fourth, observability last.

Dependencies:

- A working AWS account and region.
- Meaningful naming conventions for Route 53, CloudFront, EC2, RDS, and SES resources.
- Team agreement that this workshop documents deployed AWS behavior rather than local development setup.

### 4.1 Workshop Overview and Service Map

![AWS Workshop Architecture Overview](/images/4-Workshop/ev-achi.png)

What this part does:

- Defines the end-to-end service map of the EV Charging Station Management System.
- Separates public-facing services from private runtime and data services.
- Introduces the main request lifecycle before the reader dives into service-specific sections.

Why it exists:

- Readers need a mental model before they can troubleshoot one layer at a time.
- It frames the entire workshop as a layered system rather than a flat list of AWS products.
- It confirms the architectural stance that the backend is ALB plus EC2 rather than API Gateway plus Lambda.

Input and output behavior:

| Input or trigger | Processing in this section | Output or effect |
| --- | --- | --- |
| New reader needs orientation | The layered architecture is summarized | The reader can place each later section in context |
| Team needs a service-to-need mapping | The AWS service map table is provided | Each service is linked to a concrete architectural responsibility |
| Team needs quick environment validation | Baseline AWS CLI commands are provided | Operators can confirm the account, region, and core resources before deeper work |

High-level explanation of the workshop commands:

- `aws sts get-caller-identity` validates that the operator is authenticated to the expected account.
- `aws route53 list-hosted-zones`, `aws cloudfront list-distributions`, `aws ec2 describe-instances`, and `aws rds describe-db-instances` are baseline existence checks for the main architecture pillars.
- These commands are intentionally read-only, which makes section `4.1` a safe pre-flight validation stage.

Dependencies:

- Route 53 hosted zone exists.
- CloudFront distribution exists and is tied to the expected domain.
- EC2 instances and RDS instance exist in the target account and region.
- The workshop's naming convention matches the actual environment.

### 4.2 Edge Delivery and Geospatial Requests

#### Amazon Route 53

![Amazon Route 53 DNS](/images/4-Workshop/route53.jpg)

#### Amazon CloudFront

![Amazon CloudFront Distribution](/images/4-Workshop/cloudfront.jpg)

#### Amazon S3

![Amazon S3 Bucket](/images/4-Workshop/s3-bucket.jpg)

#### Amazon Location Service

![Amazon Location Service](/images/4-Workshop/location-service.png)

What this part does:

- Handles the public entry path for browsers.
- Serves static frontend assets with global caching and a private S3 origin.
- Protects the edge path with WAF before requests reach the origin.
- Separates map and place-search traffic from ordinary static-asset delivery.

Why it exists:

- Frontend delivery has very different scaling and latency characteristics from backend API execution.
- Static assets should be globally cached and cheap to serve, while the backend should process only dynamic traffic.
- Geospatial requests rely on a specialized AWS service and should not be forced through the same path as static files.

Service interaction flow:

1. A browser resolves `${APP_DOMAIN}` through Route 53.
2. Route 53 directs the request to CloudFront.
3. CloudFront evaluates WAF rules before serving or forwarding the request.
4. On a cache hit, CloudFront returns the object immediately.
5. On a cache miss, CloudFront reads the object from S3 through Origin Access Control.
6. After the SPA loads, the frontend calls Amazon Location Service for map rendering or place search.
7. Later API requests leave the edge module and move to the ALB-based application layer.

Input and output behavior:

| Input | Processing path | Output |
| --- | --- | --- |
| DNS lookup for `${APP_DOMAIN}` | Route 53 resolves the alias or CNAME | CloudFront hostname is returned to the client |
| HTTPS request for `index.html`, JS, CSS, or media | CloudFront serves from cache or fetches from S3 | Browser receives static assets with CDN acceleration |
| Untrusted HTTP request pattern | WAF inspects the request before origin access | Request is allowed, blocked, counted, or challenged depending on policy |
| Place search or map request | Frontend calls Amazon Location Service | Browser receives geospatial search results or map content |

High-level explanation of the workshop commands and samples:

- The hosted-zone inspection command validates that public DNS ownership is present.
- The `route53-app-record.json` sample teaches idempotent DNS management through `UPSERT`, which is a safe pattern for infrastructure updates.
- `aws s3 sync ./dist s3://${STATIC_BUCKET} --delete` models a frontend deployment pipeline in simplified form.
- The CloudFront invalidation command explains why object replacement in S3 is not enough when CDN caches are active.
- The Location Service search command proves that the map integration path is separate from both S3 and ALB.
- The S3 bucket policy sample is the key security artifact of this module because it forces frontend users through CloudFront rather than allowing direct public bucket reads.

Dependencies:

- Hosted zone and public application record.
- CloudFront distribution with an attached certificate and WAF association.
- Private S3 bucket with OAC-based read permission.
- Place index and map configuration in Amazon Location Service.
- Deployment discipline that invalidates cached assets after frontend release.

### 4.3 Private Application Routing and Compute

#### Amazon VPC

![Amazon VPC](/images/4-Workshop/vpc-overview.png)

#### Internet Gateway

![Internet Gateway](/images/4-Workshop/internet-gateway.png)

#### NAT Gateway

![NAT Gateway](/images/4-Workshop/nat-gateway.png)

#### Application Load Balancer

![Application Load Balancer](/images/4-Workshop/alb.png)

#### Amazon EC2

![Amazon EC2 Instances](/images/4-Workshop/ec2-instances.png)

#### EC2 Auto Scaling

![Auto Scaling Group](/images/4-Workshop/auto-scaling-group.png)

What this part does:

- Moves dynamic user traffic from the edge into private application compute.
- Defines how public and private subnets interact inside the VPC.
- Ensures that only the ALB is internet-facing while the EC2 runtime stays private.
- Uses Auto Scaling and health checks to keep the backend horizontally scalable and self-healing.

Why it exists:

- Dynamic APIs need controlled routing, health evaluation, and horizontal scale.
- Private subnets reduce exposure of the application servers.
- Outbound internet access is still necessary for package retrieval, AWS public endpoints, or third-party integrations, so NAT is part of the design.

Service interaction flow:

1. The frontend sends an HTTPS API request to the backend entry point.
2. The ALB receives the request on a public listener.
3. Listener rules and target-group health decide which EC2 instance should receive the request.
4. A healthy EC2 instance in a private subnet processes the business logic.
5. If the application needs outbound access, traffic exits through NAT Gateway and the Internet Gateway path.
6. The application returns the response to the ALB, which returns it to the client.
7. Auto Scaling replaces unhealthy instances or adds capacity when load increases.

Input and output behavior:

| Input | Processing path | Output |
| --- | --- | --- |
| HTTPS API request from frontend | ALB listener routes to healthy target | Backend business response reaches the caller |
| Health-check probe from ALB | EC2 health endpoint responds | Target remains healthy or is removed from rotation |
| Outbound dependency call from private EC2 | Private route table sends egress to NAT Gateway | Instance reaches external endpoints without public IP exposure |
| Capacity or failure event | Auto Scaling launches or replaces instances | Desired instance count and AZ coverage are maintained |

High-level explanation of the workshop commands and samples:

- `describe-load-balancers` confirms that the ALB exists, is internet-facing, and uses the expected subnets.
- `describe-target-health` is the most important runtime verification command in this module because it shows whether requests can actually reach healthy backend targets.
- `describe-auto-scaling-groups` reveals desired capacity, subnet placement, and lifecycle state for backend instances.
- `describe-instances` supports instance-level inspection when the team needs to compare one target against the group baseline.
- The security-group and route-table table is the core architecture logic of this section because it defines who can talk to whom.
- The recommended health-check settings teach that liveness must stay lightweight and should not fail simply because the database is under stress.

Dependencies:

- VPC with at least one public subnet and at least two private subnets for resilient application placement.
- ALB listener, target group, and health-check endpoint.
- EC2 launch template, AMI, instance profile, and security groups.
- NAT Gateway and correct route-table wiring for private-subnet egress.
- Application behavior that supports stateless or low-session-affinity scale-out.

### 4.4 Relational Data Resilience and Network Isolation

#### Amazon RDS

![Amazon RDS](/images/4-Workshop/rds-overview.png)

#### Primary Database Instance

![RDS Primary Instance](/images/4-Workshop/rds-primary.png)

What this part does:

- Describes the relational system of record used by the application.
- Establishes that the database tier is private and reachable only from approved internal resources.
- Introduces a primary and standby posture to improve resilience during database or infrastructure failure.

Why it exists:

- EV charging workflows such as user accounts, bookings, charging sessions, payments, and operator records require transactional consistency.
- Relational storage is a better fit than object or key-value storage for the workshop's stated workload.
- The data tier must remain isolated because it is the highest-value persistence boundary in the platform.

Architectural interpretation:

- The workshop chapter itself models RDS as a primary and standby design, which implies a Multi-AZ or equivalent high-availability posture.
- This is more resilient than the earlier proposal chapter, which mentioned Single-AZ SQL Server.
- For this architecture document, the `4-Workshop` chapter is treated as the newer and more operationally mature source of truth.

Service interaction flow:

1. The EC2 application retrieves database connection details from Secrets Manager.
2. The application opens a private network connection to the RDS endpoint.
3. The primary database instance handles transactional reads and writes.
4. RDS replicates state to the standby environment according to the chosen availability mode.
5. If the primary becomes unavailable, RDS initiates failover and preserves endpoint continuity as much as possible.
6. The application reconnects by using the managed RDS endpoint rather than an instance-specific address.

Input and output behavior:

| Input | Processing path | Output |
| --- | --- | --- |
| SQL read or write request from EC2 | Request reaches the primary RDS endpoint over private networking | Application receives query results or transaction acknowledgement |
| Database state change | RDS synchronizes to standby infrastructure | Higher resilience and faster recovery after failure |
| Primary failure or maintenance event | Managed failover path is triggered | Service continuity through a standby role transition |
| Unauthorized internet request | No public access path exists | Database remains unreachable from the public internet |

High-level explanation of the workshop commands and samples:

- `describe-db-instances` is the structural inspection command for this module.
- The `--query` example that returns status, `MultiAZ`, endpoint, and port is especially valuable because it proves whether the architecture matches the resilience claims in the document.
- The `reboot-db-instance --force-failover` sample is a controlled resilience drill, not a routine admin action.
- The secret JSON sample models a proper separation between application code and connection details.
- The configuration table reinforces that the RDS DNS endpoint, not an instance IP, is the correct application dependency.

Dependencies:

- Private DB subnet group across multiple subnets.
- Security-group rule from the application tier only.
- Backup retention, parameter groups, and maintenance policy aligned with the production objective.
- Application connection pooling and reconnect logic that tolerate brief failover events.

### 4.5 Operational Access and Secret Governance

#### AWS Systems Manager

![AWS Systems Manager Session Manager](/images/4-Workshop/ssm-session-manager.png)

#### IAM Instance Profile

![IAM Instance Profile](/images/4-Workshop/iam-instance-profile.png)

#### Amazon EC2 Runtime Identity

![EC2 Instance IAM Role](/images/4-Workshop/ec2-iam-role.png)

What this part does:

- Defines how humans and machines gain controlled access to a private environment.
- Replaces SSH key distribution and public bastion exposure with Systems Manager and IAM-based access.
- Moves sensitive configuration out of application files and into managed secret storage.

Why it exists:

- Production workloads should not depend on hardcoded credentials or public administration paths.
- Machine identity must be temporary, scoped, and automatically rotated.
- Secret storage and encryption should be delegated to managed AWS services instead of being reimplemented in the application layer.

Service interaction flow:

1. An administrator initiates access through Systems Manager.
2. The target EC2 instance authenticates through its attached instance profile.
3. The application or operator retrieves secrets from Secrets Manager by secret name.
4. KMS protects the underlying secret material and authorizes decryption for approved principals.
5. The same instance role can also publish logs and send email where necessary.

Input and output behavior:

| Input | Processing path | Output |
| --- | --- | --- |
| Admin wants shell access to a private instance | Session Manager session is established | Secure remote access without public SSH |
| EC2 boot or runtime AWS API call | Instance profile provides temporary credentials | Application can authenticate to AWS APIs safely |
| Application needs a credential or token | Secrets Manager returns the secret value after authorization | Runtime configuration is loaded without embedding secrets in code |
| Secret retrieval touches encrypted data | KMS enforces key-level permissions | Only approved roles can decrypt protected secret material |

High-level explanation of the workshop commands and samples:

- `aws ssm start-session` proves that the operations path does not require a bastion host.
- The EC2 instance-profile inspection command confirms whether the expected runtime identity is actually attached.
- `aws secretsmanager get-secret-value` is the most direct validation of secret naming, access policy, and region alignment.
- `aws kms list-aliases` is not the full KMS audit story, but it is a practical way to verify that the intended key naming structure exists.
- The minimum IAM policy sample captures the workshop's machine-permission philosophy: read secrets, decrypt, write logs, and send email, all without storing static AWS keys on the host.

Dependencies:

- EC2 instances must be registered and healthy in Systems Manager.
- Instance profiles need correct trust policies and least-privilege permissions.
- Secret naming, environment scoping, and KMS-key ownership must be consistent.
- If the environment is highly locked down, VPC endpoints for Secrets Manager, SSM, and CloudWatch may be preferable to internet egress through NAT.

### 4.6 Monitoring, Audit, and Email Delivery

#### Amazon CloudWatch

![Amazon CloudWatch Logs and Metrics](/images/4-Workshop/cloudwatch.png)

#### AWS CloudTrail

![AWS CloudTrail Events](/images/4-Workshop/cloudtrail.png)

#### Amazon SES

![Amazon SES](/images/4-Workshop/ses.png)

#### Route 53 Email DNS Records

![Route 53 SES DNS Records](/images/4-Workshop/route53-ses.png)

What this part does:

- Defines how the platform is observed during runtime.
- Records AWS control-plane activity for governance and security investigations.
- Sends application-originated mail through SES with Route 53-backed domain verification.

Why it exists:

- A production system is incomplete if it cannot be observed, audited, and trusted to deliver notifications.
- Email delivery is often a business-critical feature for account events, booking updates, or operational alerts.
- Audit visibility matters because the workshop relies heavily on AWS-native administrative paths such as SSM, IAM, and infrastructure APIs.

Service interaction flow:

1. The application writes logs and metrics to CloudWatch.
2. Operators inspect logs, alarms, and metrics to diagnose production behavior.
3. AWS services record control-plane activity in CloudTrail.
4. The EC2 workload sends mail through SES with its IAM-backed permissions.
5. SES depends on Route 53 records for identity verification, DKIM, SPF, and mail routing support.

Input and output behavior:

| Input | Processing path | Output |
| --- | --- | --- |
| Application log event or metric | CloudWatch stores the telemetry | Operators can search, alarm, and dashboard the signal |
| Administrative AWS API action | CloudTrail records the event | Teams can audit who changed what and when |
| Outbound email request | SES processes the message if the identity is verified | Recipient receives the notification and the platform avoids self-managed SMTP |
| SES domain verification requirement | Route 53 publishes TXT, CNAME, and MX records | SES identity becomes verifiable and deliverability improves |

High-level explanation of the workshop commands and samples:

- `aws logs tail` is the fastest live-troubleshooting command in the chapter because it shortens the feedback loop during incidents.
- `aws cloudtrail lookup-events` links operational actions to audit evidence, which is especially important when changes happen outside application code.
- `aws sesv2 get-email-identity` verifies the readiness of the mail path before the application starts sending.
- The `ses-test-email.json` payload is a minimal transactional mail contract that proves the role, identity, and region are aligned.
- `put-metric-alarm` demonstrates that observability is not only about reading logs but also about turning signals into operator-visible alerts.
- The SES Route 53 JSON sample is a concrete example of why email delivery is partly an infrastructure concern, not only an application concern.

Dependencies:

- CloudWatch log groups, retention settings, and permissions.
- CloudTrail enabled with an audit retention strategy that matches compliance needs.
- Verified SES identities and correct DNS records in Route 53.
- Application code or agent behavior that emits useful logs, metrics, and message content.

## Request Lifecycle and Service Interaction Flow

### Lifecycle 1: Static frontend delivery

1. A user enters the application domain in a browser.
2. Route 53 resolves the name to CloudFront.
3. CloudFront terminates HTTPS and applies WAF controls.
4. CloudFront returns cached assets or reads them from S3 through OAC.
5. The browser boots the frontend application.

Why this lifecycle matters:

- It minimizes latency for globally distributed users.
- It keeps the S3 bucket private.
- It reduces unnecessary dynamic traffic hitting the backend.

### Lifecycle 2: Geospatial experience

1. The SPA requests map tiles, place lookup, or search suggestions.
2. The request is sent to Amazon Location Service rather than to the application EC2 layer.
3. The frontend renders map context or search results for the user.

Why this lifecycle matters:

- It keeps geospatial workload specialized and decoupled from core transaction APIs.
- It prevents the backend from becoming an unnecessary proxy for map data.

### Lifecycle 3: Dynamic API request

1. The frontend calls the backend endpoint exposed through the ALB.
2. The ALB selects a healthy private EC2 target.
3. The EC2 application loads configuration or secrets if needed.
4. The application executes business logic and talks to RDS.
5. The application emits logs and returns the response through the ALB.

Why this lifecycle matters:

- It is the core business path of the system.
- It concentrates public exposure at the ALB instead of at the instance layer.

### Lifecycle 4: Database-backed transaction

1. EC2 obtains the current database secret.
2. The application connects to the RDS endpoint over private networking.
3. The primary instance handles the transaction.
4. RDS maintains standby continuity.
5. If failover occurs, the application reconnects by using the managed endpoint.

Why this lifecycle matters:

- It preserves transactional integrity.
- It hides database host changes behind a stable managed endpoint.

### Lifecycle 5: Operator access and diagnostics

1. An administrator starts a Systems Manager session to a private instance.
2. The instance uses its IAM role to authenticate AWS service calls.
3. The operator reviews app state, local logs, or runtime configuration.
4. Relevant activity is later visible in CloudTrail and CloudWatch.

Why this lifecycle matters:

- It delivers administration without public SSH.
- It keeps operational actions auditable.

### Lifecycle 6: Email and audit completion

1. A business event in the application triggers an email send.
2. The EC2 workload calls SES by using its instance role.
3. SES accepts the message when the sender identity and DNS records are valid.
4. Control-plane actions remain visible in CloudTrail.
5. Application-side success or failure signals are written to CloudWatch.

Why this lifecycle matters:

- It closes the loop between application actions, communication, and operator visibility.

## Deployment Considerations

| Deployment area | Current workshop posture | Recommended production handling |
| --- | --- | --- |
| Frontend | Manual or CLI-driven `aws s3 sync` plus CloudFront invalidation | Build versioned static assets, publish to S3, invalidate only changed paths when possible, and keep rollback-ready object versions |
| Backend compute | EC2 instances behind ALB and Auto Scaling | Use a launch template with immutable artifacts, perform rolling or instance-refresh deployments, and validate health checks before terminating old nodes |
| Database changes | RDS is the system of record, but migration tooling is not shown | Run schema migrations in a controlled release stage through SSM or a CI/CD job with backup and rollback planning |
| Secrets and config | Secret values are centralized in Secrets Manager | Separate secrets by environment, rotate on schedule, and avoid mixing static configuration with credentials |
| Certificates and DNS | HTTPS is implied and Route 53 is in scope | Manage CloudFront and ALB certificates with ACM and automate DNS updates carefully to reduce cutover risk |
| Operator procedures | The chapter uses CLI inspection heavily | Convert repeated CLI checks into runbooks, Infrastructure as Code validations, and dashboards |

Recommended deployment sequence:

1. Build and test the frontend artifact.
2. Publish the frontend to S3 and invalidate the required CloudFront paths.
3. Build the backend artifact and update the EC2 launch template or machine image.
4. Roll the Auto Scaling Group gradually while monitoring ALB target health and CloudWatch logs.
5. Run database migrations only after backup posture and rollback steps are confirmed.
6. Validate SES identity status, alarms, and operational access before declaring the release healthy.

## Scalability and Fault Tolerance

| Concern | Current workshop posture | Benefit | Current gap or risk | Recommended hardening |
| --- | --- | --- | --- | --- |
| Static content scale | CloudFront plus S3 | Edge caching absorbs global read traffic efficiently | Cache invalidation discipline is required after releases | Use versioned assets and tuned cache policies |
| Application scale | ALB plus Auto Scaling across multiple AZs | Horizontal scale and instance replacement are built in | Session-heavy behavior could reduce scale efficiency | Keep application nodes stateless and tune scaling policies |
| Database resilience | RDS primary and standby posture | Managed failover improves durability and recovery | Actual Multi-AZ mode must be confirmed in the real environment | Validate Multi-AZ, backup recovery, and failover drills regularly |
| Private egress | NAT Gateway for outbound traffic | Instances stay private while still reaching external services | A single NAT Gateway is a cost and availability trade-off | Use one NAT Gateway per AZ for higher resilience or add VPC endpoints to reduce NAT dependence |
| Secret access | Secrets Manager and KMS | Centralized secret governance and encryption | Overly broad IAM permissions can weaken the model | Narrow resource scopes and enable rotation |
| Operator access | Systems Manager | No bastion host or SSH key sprawl | SSM readiness depends on agent health and IAM correctness | Enable session logging and test break-glass procedures |
| Observability | CloudWatch and CloudTrail | Logs, metrics, and audit events stay centralized | Alarm coverage and retention policies are not shown | Add dashboards, alarm routing, retention classes, and metric filters |
| Email delivery | SES plus DNS verification | Transactional email is externalized to a managed service | Bounce handling and complaint workflows are not visible | Add feedback processing and sending reputation monitoring |
| Async workload isolation | Not present | Simpler architecture today | Email, retries, or side effects may remain tightly coupled to request latency | Introduce EventBridge, SQS, or SNS for background processing when needed |

## Improvement Opportunities and AWS Well-Architected Alignment

| AWS Well-Architected pillar | Current strengths in the workshop | Recommended next step |
| --- | --- | --- |
| Operational Excellence | The chapter already encourages CLI verification, environment variables, and clear service ownership | Move the documented checks into Infrastructure as Code, automated health validation, and reusable operational runbooks |
| Security | Private EC2 and RDS, WAF, SSM, IAM instance profiles, Secrets Manager, and KMS create a strong baseline | Add tighter IAM scoping, session logging, secret rotation, VPC endpoints for sensitive service access, and explicit certificate management |
| Reliability | ALB health checks, Auto Scaling, and an RDS standby posture provide meaningful resilience | Confirm database HA mode, test failover routinely, remove single-AZ NAT dependency, and consider asynchronous decoupling for non-critical side effects |
| Performance Efficiency | CloudFront caching and scale-out EC2 compute support the primary web workload well | Tune cache policies, add database connection pooling, and consider read-optimization layers if usage grows sharply |
| Cost Optimization | S3, CloudFront, and managed services reduce some operational burden | Review NAT cost, right-size EC2 and RDS, use lifecycle policies, and replace avoidable public-endpoint traffic with VPC endpoints where possible |
| Sustainability | Elastic services and managed components avoid unnecessary always-on bespoke infrastructure | Schedule non-production shutdowns, optimize cache hit ratios, and right-size long-running compute regularly |

### Natural Evolution Paths

- Introduce API Gateway only when the platform needs public API governance features that ALB does not naturally provide.
- Introduce Lambda for utility automations such as light post-processing, SES event handling, or secret-rotation helpers.
- Introduce Step Functions when booking, charging, payment, or compliance workflows become long-running and stateful.
- Introduce EventBridge, SQS, or SNS when notification, retry, or background processing should no longer be coupled to request latency.
- Introduce read replicas, Aurora, or a reporting store only when relational throughput or workload isolation requires them.

## Operational Readiness Checklist

- Confirm the public path is healthy: Route 53 resolves the expected domain, CloudFront presents the right certificate, WAF is attached, and S3 remains private behind OAC.
- Confirm the application path is healthy: ALB listeners and target groups are green, Auto Scaling spans the intended Availability Zones, and private EC2 instances can reach required dependencies.
- Confirm the data path is healthy: the RDS endpoint is private, backups are enabled, failover posture is understood, and the application reads credentials from Secrets Manager rather than local files.
- Confirm the control plane is healthy: SSM access works without SSH, IAM roles are least-privilege, KMS policies are scoped correctly, and CloudTrail records administrative actions.
- Confirm observability is healthy: CloudWatch log groups exist with retention settings, important alarms have owners, SES identities are verified, and Route 53 contains the supporting email DNS records.

## Notes / Assumptions

- This architecture document is grounded in the workshop source tree under `content/4-Workshop/`, not in the backend application repository.
- The workshop describes a more resilient RDS posture than the earlier proposal chapter. This document follows the workshop chapter as the current architecture target.
- The visible implementation is intentionally AWS-native and mostly synchronous.
- API Gateway, Lambda, ECS, EKS, DynamoDB, Step Functions, EventBridge, SQS, SNS, and Cognito are not part of the confirmed current architecture, but several of them are reasonable future additions as the system grows.
- Security groups, ACM certificates, DB subnet groups, launch templates, and agent-level integrations are treated here as required inferred components because the documented AWS flows would not operate correctly without them.
