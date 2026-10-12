---
title : "Architecture"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### System architecture

The diagram below shows the request flow and how the AWS services connect.

```
                                      (1) DNS lookup
          +-----------+   pvpwork.com  +--------------+
          |  Visitor  | -------------> |  Route 53    |
          |  browser  | <------------- |  hosted zone |
          +-----------+   alias ->     +--------------+
                |             CloudFront
                | (2) HTTPS request (TLS cert from ACM)
                v
          +---------------------------+        +--------------------+
          |      Amazon CloudFront    |  uses  | ACM certificate    |
          |  (CDN, edge locations)    |------->| (us-east-1)        |
          |  - redirect HTTP -> HTTPS |        | pvpwork.com + www  |
          |  - www -> apex (function) |        +--------------------+
          +---------------------------+
                | (3) cache miss: fetch origin
                |     signed with OAC (SigV4)
                v
          +---------------------------+
          |        Amazon S3          |  (private bucket, no public access)
          |  pvpwork-fcaj-landing     |
          |  index.html, assets/, docs/
          +---------------------------+
                ^
                | bucket policy: allow s3:GetObject
                | only from this CloudFront distribution
```

#### How a request flows

1. **DNS resolution** — the visitor's browser looks up `pvpwork.com`. Route 53 answers with an **alias record** that points directly at the CloudFront distribution (both A/IPv4 and AAAA/IPv6).
2. **HTTPS delivery** — the browser connects to the nearest CloudFront edge location over HTTPS. CloudFront presents the ACM certificate for `pvpwork.com`. Any plain HTTP request is answered with a 301 redirect to HTTPS. A CloudFront **viewer-request function** redirects `www.pvpwork.com` to the apex `pvpwork.com`.
3. **Origin fetch** — if the requested file is not already cached at the edge, CloudFront fetches it from the S3 origin. The request is signed with the **Origin Access Control (OAC)** using SigV4, so S3 can verify it comes from this exact distribution.
4. **Private storage** — the S3 bucket blocks all public access. Its bucket policy allows `s3:GetObject` **only** when the request comes from this CloudFront distribution ARN. Nobody can reach the files through the S3 URL directly.

#### Component responsibilities

| Component | Responsibility |
| --- | --- |
| **Route 53** | Authoritative DNS for the domain; alias records to CloudFront |
| **CloudFront** | Global CDN, HTTPS termination, HTTP→HTTPS redirect, www→apex redirect, caching |
| **ACM** | Free TLS certificate for apex + www (in `us-east-1`) |
| **S3** | Private origin storage for the website files |
| **OAC + bucket policy** | Restrict S3 reads to this CloudFront distribution only |

#### Security notes

+ The S3 bucket has **Block Public Access** enabled. There is no public bucket URL and no object ACLs granting public read.
+ CloudFront reaches S3 through **OAC** (the modern replacement for Origin Access Identity), scoped by the distribution ARN in the bucket policy's `AWS:SourceArn` condition.
+ The viewer protocol policy is **redirect-to-HTTPS**, and the minimum TLS version is **TLSv1.2_2021**.
