# Saroj Behera — DevOps Portfolio Live on AWS

Live URL: https://sarojops.cloud

Personal portfolio website hosted on AWS using production-grade
static website architecture with S3, CloudFront, ACM, and Cloudflare DNS.

---

## Architecture Overview

### Original Architecture (Phase 1)
```
Browser → Route 53 (DNS) → CloudFront (CDN + HTTPS) → S3 (Private Storage)
                ↑
        ACM SSL Certificate (us-east-1)
```

### Current Architecture (Phase 2 — Cost Optimized)
```
Browser → Cloudflare DNS (Free) → CloudFront (CDN + HTTPS) → S3 (Private Storage)
                                          ↑
                                ACM SSL Certificate (us-east-1)
```

---

## AWS Services Used

| Service | Region | Purpose |
|---|---|---|
| S3 | ap-south-1 (Mumbai) | Private static file storage |
| CloudFront | Global | CDN, HTTPS termination, caching |
| ACM | us-east-1 (N. Virginia) | Free SSL certificate, auto-renewing |
| ~~Route 53~~ | ~~Global~~ | ~~DNS management~~ → Replaced by Cloudflare |

---

## Phase 2 — Cloudflare Migration (April 2026)

### Why Migrated to Cloudflare
- Route 53 Hosted Zone costs $0.50/month = $6/year
- Route 53 DNS query charges on top
- Cloudflare DNS is completely free with better performance
- Cloudflare provides free SSL, DDoS protection, and CDN features

### Cost Savings
| Service | Before | After |
|---|---|---|
| Route 53 Hosted Zone | $0.50/month | $0 ✅ |
| Route 53 DNS Queries | Charged per million | $0 ✅ |
| Cloudflare DNS | - | Free forever ✅ |
| Cloudflare SSL | - | Free forever ✅ |
| **Total Saved** | **~$6-8/year** | **$0** ✅ |

### Migration Steps Performed

#### Step 1 — Added Domain to Cloudflare
- Added sarojops.cloud to Cloudflare (Free plan)
- Cloudflare auto-imported existing DNS records

#### Step 2 — Cleaned Up DNS Records in Cloudflare
- Deleted 4 old A records (52.222.177.x) that were auto-imported from Route 53
- These were old CloudFront edge IPs — not needed with CNAME approach

#### Step 3 — Created CNAME Records in Cloudflare
| Type | Name | Target | Proxy Status |
|---|---|---|---|
| CNAME | @ (root) | d2nv968btkkj7r.cloudfront.net | DNS Only |
| CNAME | www | d2nv968btkkj7r.cloudfront.net | DNS Only |

> DNS Only (grey cloud) used — required for CloudFront SSL to work correctly

#### Step 4 — Updated Nameservers in GoDaddy
- Domain registered at GoDaddy
- Replaced default nameservers with Cloudflare nameservers:
  - `aitana.ns.cloudflare.com`
  - `emerson.ns.cloudflare.com`

#### Step 5 — Deleted Route 53 Hosted Zone
- Deleted all DNS records inside the hosted zone
- Deleted the hosted zone itself
- Stops $0.50/month charge immediately

#### Step 6 — Configured Cloudflare SSL
- SSL/TLS mode set to **Full**
- Encrypts traffic: Browser → Cloudflare → CloudFront (origin)
- Free SSL via Cloudflare — no certificate management needed on this layer

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

### Cloudflare DNS (Replaced Route 53)
- CNAME record at root domain pointing to CloudFront distribution
- DNS Only mode (not proxied) — preserves CloudFront SSL behavior
- Free plan — no query charges, no monthly fee
- Nameservers managed via GoDaddy registrar

---

## Security Model
```
User → https://sarojops.cloud (Cloudflare SSL + CloudFront HTTPS) → S3 private bucket
User → https://s3.ap-south-1.amazonaws.com/sarojops.cloud/index.html → 403 Access Denied
```

Direct S3 URL access returns 403 — proving files are
only accessible through CloudFront.

---

## Project Structure
```
Project-Saroj-Profile-LiveDomain/
│
├── index.html          # Main portfolio page
└── README.md           # This file
```

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
- DNS validation — create CNAME records in Cloudflare
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

### 5. Cloudflare DNS Setup
- Add domain to Cloudflare (Free plan)
- Delete any auto-imported A records for root domain
- Add CNAME record: @ → CloudFront distribution domain (DNS Only)
- Add CNAME record: www → CloudFront distribution domain (DNS Only)
- Update nameservers in GoDaddy to Cloudflare nameservers
- Set SSL/TLS mode to Full in Cloudflare

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

### Phase 1 (Original)
| Service | Approx Cost |
|---|---|
| S3 storage + requests | ~$0.01/month |
| CloudFront | ~$0.50/month |
| Route 53 hosted zone | $0.50/month |
| ACM certificate | Free |
| **Total** | **~$1/month** |

### Phase 2 (Current — Cost Optimized)
| Service | Approx Cost |
|---|---|
| S3 storage + requests | ~$0.01/month |
| CloudFront | ~$0.50/month |
| Route 53 hosted zone | $0 (deleted) ✅ |
| Cloudflare DNS | Free ✅ |
| Cloudflare SSL | Free ✅ |
| GoDaddy domain renewal | ~$1.5/month (~$18/year) |
| ACM certificate | Free |
| **Total** | **~$0.51/month** |

---

## Author

Saroj Behera — DevOps Engineer
- Website: https://sarojops.cloud
- GitHub: https://github.com/sarojbehera0610