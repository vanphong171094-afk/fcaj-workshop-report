---
title : "Create and upload to S3"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

In this step you create the Route 53 hosted zone that will hold your DNS, then create a private S3 bucket and upload the website files.

#### Step 1 — Create the Route 53 hosted zone

The hosted zone must exist first, because the ACM certificate (next step) is validated using DNS records placed in this zone.

```bash
aws route53 create-hosted-zone \
  --name pvpwork.com \
  --caller-reference "pvpwork-$(date +%s)" \
  --hosted-zone-config Comment="FCAJ landing page"
```

The response includes the **hosted zone Id** (e.g. `Z0036429187B89PBOPZ1R`) and four **NameServers**. Note both.

{{% notice info %}}
Go to your domain registrar and replace the current nameservers with the four Route 53 nameservers from the response. For a `.com` domain, propagation usually takes a few minutes to a few hours. This step is manual and done outside AWS.
{{% /notice %}}

Verify the nameservers once they have propagated:

```bash
dig +short NS pvpwork.com @8.8.8.8
```

The output should list the four `awsdns` nameservers, not your old registrar's.

#### Step 2 — Create a private S3 bucket

Create the bucket in the Singapore region (`ap-southeast-1`), close to visitors in Vietnam. New buckets have **Block Public Access** on by default — keep it that way.

```bash
aws s3api create-bucket \
  --bucket pvpwork-fcaj-landing \
  --region ap-southeast-1 \
  --create-bucket-configuration LocationConstraint=ap-southeast-1
```

{{% notice note %}}
S3 bucket names are globally unique. If `pvpwork-fcaj-landing` is taken, choose another name and use it consistently in the following steps.
{{% /notice %}}

#### Step 3 — Upload the website files

Use `aws s3 sync` to upload the whole folder. Exclude local junk files and anything you do not want published.

```bash
aws s3 sync /path/to/fcaj-landing s3://pvpwork-fcaj-landing \
  --exclude ".DS_Store" \
  --exclude "*/.DS_Store" \
  --exclude "README.txt" \
  --delete
```

+ `--delete` removes objects in the bucket that no longer exist locally, keeping the bucket in sync.
+ The upload includes `index.html`, everything under `assets/`, and the PDFs under `docs/`.

Confirm the files are present:

```bash
aws s3 ls s3://pvpwork-fcaj-landing --recursive
```

{{% notice tip %}}
The bucket is private, so you **cannot** open these files from an S3 URL in a browser yet. They become reachable only through CloudFront, which you set up in step 5.6.
{{% /notice %}}
