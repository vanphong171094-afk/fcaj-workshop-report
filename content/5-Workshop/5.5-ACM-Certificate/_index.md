---
title : "Request an ACM certificate"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

In this step you request a free TLS/SSL certificate for the domain and validate it with DNS.

{{% notice warning %}}
A certificate used by CloudFront **must be in the `us-east-1` region** (N. Virginia), regardless of where your S3 bucket is. This is the most common mistake in this workshop.
{{% /notice %}}

#### Step 1 — Request the certificate

Request one certificate covering both the apex domain and the `www` subdomain, validated by DNS.

```bash
aws acm request-certificate \
  --region us-east-1 \
  --domain-name pvpwork.com \
  --subject-alternative-names www.pvpwork.com \
  --validation-method DNS \
  --key-algorithm RSA_2048 \
  --tags Key=Project,Value=fcaj-landing
```

The response returns the **certificate ARN**, e.g.
`arn:aws:acm:us-east-1:669167565740:certificate/e7fd36b9-...`. Note this ARN.

#### Step 2 — Get the DNS validation records

ACM generates one CNAME record per domain name. Retrieve them:

```bash
aws acm describe-certificate \
  --region us-east-1 \
  --certificate-arn <CERT-ARN> \
  --query 'Certificate.DomainValidationOptions[].ResourceRecord'
```

#### Step 3 — Add the validation records to Route 53

Create a change-batch file (`acm-validation.json`) with the Name/Value pairs from the previous step:

```json
{
  "Comment": "ACM DNS validation for pvpwork.com",
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "_xxxx.pvpwork.com.",
        "Type": "CNAME",
        "TTL": 300,
        "ResourceRecords": [ { "Value": "_yyyy.acm-validations.aws." } ]
      }
    }
  ]
}
```

Apply it to your hosted zone:

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id Z0036429187B89PBOPZ1R \
  --change-batch file://acm-validation.json
```

#### Step 4 — Wait for the certificate to be issued

ACM validates automatically once it finds the DNS records. Check the status:

```bash
aws acm describe-certificate \
  --region us-east-1 \
  --certificate-arn <CERT-ARN> \
  --query 'Certificate.Status'
```

The status moves from `PENDING_VALIDATION` to `ISSUED`, usually within a few minutes.

{{% notice warning %}}
If the status is **stuck on `PENDING_VALIDATION`**, it almost always means the domain's nameservers **do not point to Route 53 yet**. ACM validates by resolving the records over public DNS; if the registrar still serves the old nameservers, ACM never sees the validation records even though they are correctly placed in Route 53. Finish the nameserver change in step 5.4 first.
{{% /notice %}}
