---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

{{% notice warning %}}
⚠️ **Note:** The information below is for reference purposes only. Please **do not copy verbatim** for your report, including this warning.
{{% /notice %}}

In this section, I summarize the project I planned to deliver during the program.

# FCAJ Landing Page on AWS
## A serverless, pay-per-use static website on a custom domain

### 1. Executive Summary
The project publishes the First Cloud AI Journey (FCAJ) landing page to the public Internet on the custom domain `pvpwork.com`, using a fully serverless AWS architecture. The site is bilingual (Vietnamese/English), includes downloadable program PDFs, and must load quickly for visitors in Vietnam while costing almost nothing to run at low, occasional traffic.

### 2. Problem Statement
**The problem.** The FCAJ landing page existed only as local HTML files. There was no public URL, no HTTPS, no custom domain, and no fast delivery for visitors. Running a traditional web server for a purely static site would add needless cost and maintenance.

**The solution.** Host the files privately in Amazon S3 and serve them worldwide through Amazon CloudFront over HTTPS, secured by a free ACM certificate, with Amazon Route 53 pointing the custom domain at CloudFront. The S3 bucket stays private; only CloudFront can read it through an Origin Access Control (OAC).

**Benefits.** HTTPS by default, global CDN performance, no servers to patch, and a monthly cost close to the Route 53 hosted zone fee (about $0.50). The same setup scales from a handful of visitors to very high traffic with no change.

### 3. Solution Architecture
A static website delivered through a CDN, with private origin storage:

+ **Amazon S3** — private bucket holding `index.html`, images, and PDFs.
+ **Amazon CloudFront** — global CDN, HTTPS termination, HTTP→HTTPS redirect, and a function that redirects `www` to the apex domain.
+ **AWS Certificate Manager (ACM)** — free TLS certificate for apex + www, in `us-east-1`.
+ **Amazon Route 53** — hosted zone for `pvpwork.com`, alias records to CloudFront.
+ **OAC + bucket policy** — restrict S3 reads to the single CloudFront distribution.

Full details are in the [Workshop](../5-Workshop/) section.

### 4. Technical Implementation
**Phases**
- Design the architecture and choose regions (S3 in `ap-southeast-1`, ACM in `us-east-1`).
- Create the Route 53 hosted zone and move nameservers from the registrar (disable DNSSEC first).
- Request and DNS-validate the ACM certificate.
- Create the private S3 bucket and upload the site.
- Create CloudFront with OAC, attach the bucket policy, add the www→apex function.
- Point DNS at CloudFront with alias records and verify end-to-end.

**Requirements**: an AWS account with permissions for S3/CloudFront/ACM/Route 53, AWS CLI v2, and a registered domain.

### 5. Timeline & Milestones
- Week 10: build the full deployment on AWS.
- Week 11: custom domain, www redirect, verification, cost review.
- Week 12: documentation and workshop report.

### 6. Budget Estimation
+ **Route 53 hosted zone:** ~$0.50/month.
+ **S3 storage + requests:** a few cents/month for ~1.5 MB of assets.
+ **CloudFront:** within the free tier at low traffic; a few cents beyond it.
+ **ACM certificate:** free.
+ **CloudFront Function (www redirect):** within the 2M requests/month free tier.

**Estimated total: about $0.50–$1.00/month** at low, occasional traffic.

### 7. Risk Assessment
- **DNSSEC blocking nameserver change** — medium impact, resolved by disabling DNSSEC at the registrar first.
- **ACM stuck pending** — caused by nameservers not yet pointing to Route 53; mitigated by verifying propagation before expecting issuance.
- **Accidental public exposure of S3** — mitigated by Block Public Access plus an OAC-scoped bucket policy.

### 8. Expected Outcomes
A live, HTTPS-secured landing page at `https://pvpwork.com`, served globally through CloudFront, with the `www` subdomain redirecting to the apex. A reusable deployment pattern and a one-command `deploy.sh` script for future updates.
