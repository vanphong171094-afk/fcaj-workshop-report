---
title : "Overview"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### What you will build

You will deploy a static website to AWS and make it reachable at a custom domain over HTTPS. By the end of this workshop, visiting `https://your-domain.com` returns your landing page from the nearest CloudFront edge location, and the `www` subdomain redirects to the apex domain.

In this report the real values used are:

| Item | Value |
| --- | --- |
| Domain | `pvpwork.com` (apex) and `www.pvpwork.com` |
| Region (S3) | `ap-southeast-1` (Singapore) |
| Region (ACM) | `us-east-1` (required by CloudFront) |

#### Why this architecture

A static landing page has no server-side code, so running a traditional web server would be wasteful. The S3 + CloudFront pattern gives you:

+ **Low cost** — you pay only for storage and the traffic actually served. With low, occasional traffic the bill is close to the Route 53 hosted zone cost (about **$0.50/month**).
+ **Global performance** — CloudFront caches your files at edge locations around the world, including Asia, so visitors in Vietnam are served quickly.
+ **HTTPS by default** — a free ACM certificate secures the domain, and all HTTP requests are redirected to HTTPS.
+ **Security** — the S3 bucket is never public. Only CloudFront can read it, enforced by an Origin Access Control (OAC) and a bucket policy.
+ **Scalability** — the same setup serves ten visitors or ten million without any change.

#### Key AWS services used

+ **Amazon S3** — object storage that holds the website files (HTML, images, PDFs). Configured as a private bucket.
+ **Amazon CloudFront** — a Content Delivery Network (CDN) that caches and serves the files globally over HTTPS, and is the only principal allowed to read the bucket.
+ **AWS Certificate Manager (ACM)** — issues and auto-renews a free TLS/SSL certificate for the domain. Must be in `us-east-1` to be used by CloudFront.
+ **Amazon Route 53** — DNS service. Holds the hosted zone for the domain and uses alias records to point the domain at the CloudFront distribution.
