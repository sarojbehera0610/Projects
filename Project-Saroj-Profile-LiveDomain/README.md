# Saroj Behera — DevOps Portfolio Live on AWS

Live URL: https://sarojops.cloud

Personal portfolio website hosted on AWS using production-grade 
static website architecture with S3, CloudFront, ACM, and Route 53.

---

## Architecture Overview
Browser → Route 53 (DNS) → CloudFront (CDN + HTTPS) → S3 (Private Storage)
↑
ACM SSL Certificate (us-east-1)

---

## AWS Services Used

| Service | Region | Purpose |
|---|---|---|
| S3 | ap-south-1 (Mumbai) | Private static file storage |
| CloudFront | Global | CDN, HTTPS termination, caching |
| ACM | us-east-1 (N. Virginia) | Free SSL certificate, auto-renewing |
| Route 53 | Global | DNS management, ALIAS record |

---

## Key Design Decisions

### S3 — Private Bucket with OAC
- Bucket is fully private — block all public access enabled
- ACLs disabled — bucket owner enforced
- Static website hosting disabled
- Only CloudFront can read files via OAC signed requests

### CloudFront — Origin Access Control (OAC)
- OAC replaces the old OAI (Origin Access Identity) method
- CloudFront presents a signed AWS identity to S3
- HTTP automatically redirects to HTTPS via behavior policy
- CachingOptimized policy for best static site performance
- All edge locations enabled for global performance

### ACM Certificate
- Certificate must be in us-east-1 regardless of S3 region
- This is a CloudFront hard requirement
- DNS validation used — auto-renews without manual intervention
- Covers sarojops.cloud 

### Route 53
- A record with ALIAS to CloudFront distribution
- ALIAS used instead of CNAME on root domain (DNS spec requirement)
- ALIAS to CloudFront has no extra DNS query charges

---

## Security Model
User → https://sarojops.cloud (CloudFront HTTPS) → S3 private bucket
User → https://s3.ap-south-1.amazonaws.com/sarojops.cloud/index.html
→ 403 Access Denied (OAC blocks all direct S3 access)

Direct S3 URL access returns 403 — proving files are
only accessible through CloudFront.

---

## Project Structure
Project-Saroj-Profile-LiveDomain/
│
├── index.html          # Main portfolio page
└── README.md           # This file

---

## Deployment Steps

### 1. S3 Bucket Setup
- Create bucket named sarojops.cloud in ap-south-1
- Block all public access ON
- Object ownership — Bucket owner enforced (ACLs disabled)
- Static website hosting — Disabled
- Upload index.html

### 2. ACM Certificate
- Switch console to us-east-1
- Request public certificate for sarojops.cloud 
- DNS validation — create CNAME records in Route 53
- Wait for status — Issued

### 3. CloudFront Distribution
- Origin domain — S3 REST endpoint (from dropdown)
- Origin access — Origin access control (OAC)
- Create new OAC for the bucket
- Copy bucket policy from yellow banner → apply to S3
- Viewer protocol policy — Redirect HTTP to HTTPS
- Cache policy — CachingOptimized
- Alternate domain names — sarojops.cloud 
- Custom SSL certificate — select ACM cert from us-east-1
- Default root object — index.html

### 4. S3 Bucket Policy (OAC)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipal",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::sarojops.cloud/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::ACCOUNT_ID:distribution/DIST_ID"
        }
      }
    }
  ]
}
```

### 5. Route 53 DNS
- Hosted zone — sarojops.cloud
- Create A record — Alias to CloudFront distribution

---

## Update & Redeploy Commands

When you update index.html, run these two commands:
```bash
# Sync files to S3
aws s3 sync . s3://sarojops.cloud --delete --exclude ".git/*" --exclude "README.md"

# Clear CloudFront cache so visitors see new version immediately
aws cloudfront create-invalidation --distribution-id YOUR_DIST_ID --paths "/*"
```

Then push to GitHub:
```bash
git add .
git commit -m "Update portfolio content"
git push origin main
```

---

## Verification Tests

| Test | URL | Expected Result |
|---|---|---|
| HTTPS live | https://sarojops.cloud | Portfolio loads with padlock |
| HTTP redirect | http://sarojops.cloud | Redirects to HTTPS |
| Direct S3 URL | https://s3.ap-south-1.amazonaws.com/sarojops.cloud/index.html | 403 Access Denied |

---

## Cost Estimate

| Service | Approx Cost |
|---|---|
| S3 storage + requests | ~$0.01/month |
| CloudFront | ~$0.50/month |
| Route 53 hosted zone | $0.50/month |
| ACM certificate | Free |
| Total | ~$1/month |

---

## Author

Saroj Behera — DevOps Engineer
- Website: https://sarojops.cloud
- GitHub: https://github.com/sarojbehera0610