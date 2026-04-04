---
title: "Edge Delivery and Geospatial Requests"
date: 2026-04-04
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# Hands-on Edge Delivery and Geospatial Requests

## Overview

- This section focuses on the public edge of the platform.
- It explains how DNS, CDN caching, WAF filtering, S3 origin access, and Location Service requests work together.
- It is the right place to start when the frontend is slow, the domain does not resolve, or map data is missing.

## AWS Services in This Section

### Amazon Route 53
![Amazon Route 53 DNS](/images/4-Workshop/route53.jpg)
- Resolves the public domain and directs users to the appropriate entry point.
- Acts as the first step in the request flow.

### Amazon CloudFront
![Amazon CloudFront Distribution](/images/4-Workshop/cloudfront.jpg)
- Delivers cached frontend assets with low latency.
- Reduces repeated requests to the origin (S3).

### AWS WAF
- Filters incoming HTTP/HTTPS requests.
- Protects the system from common web attacks at the edge.

### Amazon S3
- Stores static frontend assets (HTML, JS, CSS).
- Serves as the origin for CloudFront via Origin Access Control.

### Amazon Location Service
![Amazon Location Service](/images/4-Workshop/location-service.png)
- Handles geospatial requests such as map rendering and place search.
- Operates independently from the static asset delivery flow.

## Step-by-Step AWS Flow

1. A user opens `${APP_DOMAIN}` in the browser.
   Service: Amazon Route 53.
   What happens: Route 53 resolves the public hostname to the CloudFront distribution.
   Why it matters: If DNS is wrong, no later AWS service can help.
2. CloudFront receives the HTTPS request.
   Services: Amazon CloudFront and AWS WAF.
   What happens: WAF rules inspect the request before the distribution serves content.
   Why it matters: Malicious or malformed traffic can be blocked at the edge.
3. CloudFront checks whether the requested file is cached.
   Services: Amazon CloudFront and Amazon S3.
   What happens: Cached files are returned immediately; cache misses are fetched from S3.
   Why it matters: Cache hits reduce latency and lower load on the origin bucket.
4. S3 returns frontend objects only through the approved CloudFront path.
   Services: Amazon S3 and CloudFront Origin Access Control.
   What happens: The bucket stays private while CloudFront remains the allowed reader.
   Why it matters: Users cannot bypass CloudFront and read objects directly from S3.
5. The frontend later requests geospatial data.
   Service: Amazon Location Service.
   What happens: The browser or frontend code calls place index and map APIs.
   Why it matters: Map traffic is separated from static asset delivery.
6. The frontend then calls backend APIs through the application layer.
   Service: Application Load Balancer in the next section.
   What happens: Dynamic traffic leaves the edge path and enters the private compute stack.
   Why it matters: Static and dynamic traffic can be scaled and tuned independently.

## AWS CLI Walkthrough

### 1. Inspect the hosted zone

```bash
aws route53 list-hosted-zones --region ${AWS_REGION}
```

### 2. Create or update the application DNS record

Create a file named `route53-app-record.json`:

```json
{
  "Comment": "Point app subdomain to CloudFront",
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "app.example.com",
        "Type": "CNAME",
        "TTL": 300,
        "ResourceRecords": [
          {
            "Value": "d111111abcdef8.cloudfront.net"
          }
        ]
      }
    }
  ]
}
```

Apply the change and wait until it is in sync:

```bash
CHANGE_ID=$(aws route53 change-resource-record-sets \
  --hosted-zone-id ${HOSTED_ZONE_ID} \
  --change-batch file://route53-app-record.json \
  --query 'ChangeInfo.Id' \
  --output text)

aws route53 wait resource-record-sets-changed --id ${CHANGE_ID}
```

### 3. Deploy frontend assets to S3

```bash
aws s3 sync ./dist s3://${STATIC_BUCKET} --delete
```

### 4. Invalidate the CloudFront cache after deployment

```bash
aws cloudfront create-invalidation \
  --distribution-id ${CLOUDFRONT_DIST_ID} \
  --paths "/*"
```

### 5. Test a Location Service search

```bash
aws location search-place-index-for-text \
  --index-name ${PLACE_INDEX_NAME} \
  --text "EV charging station" \
  --region ${AWS_REGION}
```

## Configuration Example

### Private S3 origin policy for CloudFront OAC

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipalReadOnly",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::ev-prod-frontend/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/E123ABC456DEF"
        }
      }
    }
  ]
}
```

What this configuration achieves:

- The S3 bucket stays private.
- Only the named CloudFront distribution can read objects.
- Users must access the frontend through CloudFront, where WAF can inspect traffic.

## What to Verify

- DNS resolves to the expected CloudFront distribution.
- The S3 bucket is not publicly readable.
- A CloudFront invalidation is created after each deployment.
- Map search works through Amazon Location Service.
- WAF is associated with the CloudFront distribution in the target environment.

## Common Mistakes

- Updating S3 content but forgetting to invalidate CloudFront.
- Making the S3 bucket public instead of using OAC.
- Pointing Route 53 to the wrong CloudFront domain.
- Troubleshooting map failures in the backend when the real issue is Amazon Location configuration.

## Notes / Assumptions

- The diagram shows Amazon Location Service as a frontend integration path, but it does not expose the exact map style or place index names.
- Route 53 record type may be an alias rather than a CNAME in a real production setup. The example above uses a CNAME for clarity.
- The diagram does not show API Gateway or Lambda at the edge. Dynamic requests move to the ALB-based application layer.


