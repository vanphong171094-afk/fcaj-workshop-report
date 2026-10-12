---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

{{% notice warning %}}
⚠️ **Note:** The information below is for reference purposes only. Please **do not copy verbatim** for your report, including this warning.
{{% /notice %}}

# Deploy a Static Landing Page on AWS with S3, CloudFront, Route 53 and ACM

#### Overview

In this workshop, you will publish a static website (the **FCAJ landing page**) to the Internet on your own custom domain, using a fully serverless, pay-per-use AWS architecture. There is no web server to manage and no fixed monthly cost beyond the DNS hosted zone.

You will host the website files privately in **Amazon S3**, serve them worldwide through the **Amazon CloudFront** CDN over HTTPS, secure the domain with a free **AWS Certificate Manager (ACM)** certificate, and point your custom domain at the distribution using **Amazon Route 53**. The S3 bucket stays private: only CloudFront can read it, through an **Origin Access Control (OAC)**.

This pattern is the AWS-recommended way to host a static site: cheap, globally fast, HTTPS by default, and able to scale from zero to very high traffic without any change.

#### Content

1. [Overview](5.1-Overview/)
2. [Prerequisite](5.2-Prerequisite/)
3. [Architecture](5.3-Architecture/)
4. [Create and upload to S3](5.4-S3-Setup/)
5. [Request an ACM certificate](5.5-ACM-Certificate/)
6. [Create the CloudFront distribution](5.6-CloudFront/)
7. [Configure Route 53 DNS](5.7-Route53-DNS/)
8. [Clean up](5.8-Cleanup/)
