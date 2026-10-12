---
title : "Clean up"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

To avoid ongoing charges, remove the resources you created. Delete them in the order below, because some resources depend on others.

{{% notice warning %}}
Only clean up if you want to tear the project down. If this is a live site you intend to keep, **skip this page**. The running cost is very low (about $0.50/month for the hosted zone plus a few cents of usage).
{{% /notice %}}

#### Step 1 — Disable and delete the CloudFront distribution

A distribution must be disabled and fully deployed before it can be deleted.

```bash
# Disable: fetch the config, set Enabled=false, then update-distribution with the ETag.
# Wait until the status is Deployed again, then delete:
aws cloudfront delete-distribution --id ERM2MH804SI8I --if-match <ETAG>
```

#### Step 2 — Delete the CloudFront Function and OAC

```bash
aws cloudfront delete-function --name pvpwork-www-redirect --if-match <ETAG>
aws cloudfront delete-origin-access-control --id E2HFW5VE5I8YJO --if-match <ETAG>
```

#### Step 3 — Empty and delete the S3 bucket

```bash
aws s3 rm s3://pvpwork-fcaj-landing --recursive
aws s3api delete-bucket --bucket pvpwork-fcaj-landing --region ap-southeast-1
```

#### Step 4 — Delete the ACM certificate

```bash
aws acm delete-certificate \
  --region us-east-1 \
  --certificate-arn arn:aws:acm:us-east-1:669167565740:certificate/e7fd36b9-...
```

#### Step 5 — Delete the Route 53 records and hosted zone

First remove the alias and validation records you added (a hosted zone can only be deleted when it contains just the default NS and SOA records), then delete the zone:

```bash
# Delete the alias (A/AAAA) and ACM validation (CNAME) record sets with
# change-resource-record-sets using Action=DELETE, then:
aws route53 delete-hosted-zone --id Z0036429187B89PBOPZ1R
```

{{% notice tip %}}
If you keep the domain, you can leave the hosted zone in place and only delete CloudFront, the function, the OAC, S3, and the certificate. Deleting the hosted zone means you would have to re-create it and change nameservers again later.
{{% /notice %}}

#### Verify nothing is left

```bash
aws s3 ls | grep pvpwork-fcaj-landing            # should return nothing
aws cloudfront list-distributions                 # distribution gone
aws route53 list-hosted-zones-by-name --dns-name pvpwork.com
```
