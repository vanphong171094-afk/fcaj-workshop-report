---
title : "Create the CloudFront distribution"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

In this step you create an Origin Access Control (OAC), create the CloudFront distribution, then update the bucket policy so only this distribution can read S3.

#### Step 1 — Create an Origin Access Control (OAC)

OAC lets CloudFront sign its requests to S3 with SigV4, so the bucket can stay fully private.

```bash
aws cloudfront create-origin-access-control \
  --origin-access-control-config \
  Name=pvpwork-fcaj-oac,Description="OAC for pvpwork fcaj landing",SigningProtocol=sigv4,SigningBehavior=always,OriginAccessControlOriginType=s3
```

Note the **OAC Id** in the response (e.g. `E2HFW5VE5I8YJO`).

#### Step 2 — Create the CloudFront distribution

Prepare a `cf-config.json` file. The important settings are:

+ `Origins` points at the bucket domain: `pvpwork-fcaj-landing.s3.ap-southeast-1.amazonaws.com`, with the `OriginAccessControlId` from above.
+ `Aliases` lists `pvpwork.com` and `www.pvpwork.com`.
+ `DefaultRootObject` is `index.html`.
+ `ViewerProtocolPolicy` is `redirect-to-https`.
+ `ViewerCertificate` uses the ACM certificate ARN, `SSLSupportMethod=sni-only`, `MinimumProtocolVersion=TLSv1.2_2021`.
+ `CachePolicyId` is `658327ea-f89d-4fab-a63d-7e88639e58f6` (AWS managed "CachingOptimized" policy).
+ `PriceClass` is `PriceClass_200` (includes Asian edge locations, drops the more expensive regions to save cost).

```bash
aws cloudfront create-distribution --distribution-config file://cf-config.json
```

The response returns the **Distribution Id** (e.g. `ERM2MH804SI8I`) and the **CloudFront domain name** (e.g. `d3pj405a0bg80k.cloudfront.net`). The initial status is `InProgress`; CloudFront needs a few minutes to deploy to all edges.

{{% notice note %}}
The ACM certificate must be `ISSUED` at the time you create the distribution. If it is not, finish step 5.5 first.
{{% /notice %}}

#### Step 3 — Attach a bucket policy allowing the OAC

Now allow CloudFront to read the bucket. Create `bucket-policy.json`, allowing only this exact distribution ARN:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontOAC",
      "Effect": "Allow",
      "Principal": { "Service": "cloudfront.amazonaws.com" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::pvpwork-fcaj-landing/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::669167565740:distribution/ERM2MH804SI8I"
        }
      }
    }
  ]
}
```

```bash
aws s3api put-bucket-policy \
  --bucket pvpwork-fcaj-landing \
  --policy file://bucket-policy.json
```

#### Step 4 — Wait for the distribution to deploy

```bash
aws cloudfront get-distribution --id ERM2MH804SI8I --query 'Distribution.Status'
```

Wait until the status is `Deployed`. You can then test using the CloudFront domain, before DNS points to it:

```bash
curl -I https://d3pj405a0bg80k.cloudfront.net/
```

This should return `HTTP/2 200`.
