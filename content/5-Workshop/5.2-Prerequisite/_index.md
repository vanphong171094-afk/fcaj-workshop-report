---
title : "Prerequisite"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

Before starting, make sure you have the following.

#### 1. An AWS account with sufficient permissions

You need an IAM user or role allowed to work with S3, CloudFront, ACM and Route 53. For a personal lab account, the managed policy **`AdministratorAccess`** is the simplest. In production, scope the permissions down to only these services.

{{% notice tip %}}
Verify who you are authenticated as before you begin:
```bash
aws sts get-caller-identity
```
This returns your `Account`, `UserId` and `Arn`. If it errors, your credentials are not configured yet.
{{% /notice %}}

#### 2. AWS CLI v2 installed and configured

Install the AWS CLI v2, then configure it:

```bash
aws configure
```

Enter your Access Key ID, Secret Access Key, a default region (for example `ap-southeast-1`), and output format `json`.

{{% notice info %}}
You can create an Access Key in the AWS Console: **IAM → Users → your user → Security credentials → Create access key → Command Line Interface (CLI)**.
{{% /notice %}}

#### 3. A registered domain name

You need a domain you control, for example `pvpwork.com`. It can be registered anywhere (Route 53, or a third-party registrar such as Mắt Bão, GoDaddy, Namecheap). In this workshop the domain is registered with a third-party registrar, and we will move DNS management to Route 53.

{{% notice warning %}}
If your domain has **DNSSEC enabled** at the registrar, you must **disable DNSSEC first**. DNSSEC signs DNS records with keys tied to the current nameservers; changing nameservers while it is on breaks the chain of trust, and most registrars will block the nameserver change until DNSSEC is turned off.
{{% /notice %}}

#### 4. The website files

A folder of static files to publish. In this workshop it is the FCAJ landing page:

```
fcaj-landing/
├── index.html
├── assets/        (logo, photos, favicon — .webp / .svg)
└── docs/          (program PDFs)
```

No build step and no database are required — the site is pure static files.
