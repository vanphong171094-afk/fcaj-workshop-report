---
title: "Blogs Posted"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

{{% notice warning %}}
⚠️ **Note:** The information below is for reference purposes only. Please **do not copy verbatim** for your report, including this warning.
{{% /notice %}}

This section lists the blog posts I shared with the [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj) community during the program. Each post documents something I learned while building the capstone project.

### [Blog 1 - Hosting a static site privately with S3 + CloudFront OAC](3.1-Blog1/)
This post explains how to serve a static website from a **private** S3 bucket by placing CloudFront in front of it with an Origin Access Control (OAC). It covers why Block Public Access should stay on, how the bucket policy scopes access to a single distribution ARN, and why OAC replaced the older Origin Access Identity.

### [Blog 2 - Free HTTPS for any domain with AWS Certificate Manager](3.2-Blog2/)
This post walks through requesting a free TLS certificate in ACM, validating it with DNS records in Route 53, and the common pitfall of the certificate being stuck in `PENDING_VALIDATION` when the domain's nameservers do not yet point to Route 53. It also notes that CloudFront certificates must live in `us-east-1`.

### [Blog 3 - Pointing an apex domain at CloudFront and redirecting www](3.3-Blog3/)
This post shows how a Route 53 alias record lets an apex domain target CloudFront directly (something a plain CNAME cannot do), and how a small CloudFront Function redirects `www` to the apex for a single canonical URL, all within the free tier.
