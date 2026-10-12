# Workshop Report — Deploy a Static Landing Page on AWS

Bilingual (VI/EN) workshop report written for the FCJ workshop template
([thienluhoan/fcj-workshop-template](https://github.com/thienluhoan/fcj-workshop-template)),
a Hugo site using the `hugo-theme-learn` theme.

**Topic:** Deploy a static landing page to AWS using S3 + CloudFront + Route 53 + ACM.

## What is included

This folder contains the **Workshop** chapter (`content/5-Workshop/`) with the four
sections you asked for, plus the standard surrounding pages:

```
content/5-Workshop/
├── _index.md / _index.vi.md          # Workshop landing + Overview intro
├── 5.1-Overview/                     # Overview (Tổng quan)
├── 5.2-Prerequisite/                 # Prerequisite (Điều kiện tiên quyết)
├── 5.3-Architecture/                 # System architecture (Mô tả kiến trúc hệ thống)
├── 5.4-S3-Setup/                     # Hands-on: hosted zone + S3 bucket + upload
├── 5.5-ACM-Certificate/              # Hands-on: request + DNS-validate certificate
├── 5.6-CloudFront/                   # Hands-on: OAC + distribution + bucket policy
├── 5.7-Route53-DNS/                  # Hands-on: alias records + www→apex + verify
└── 5.8-Cleanup/                      # Clean up (dọn dẹp)
```

Each section has an English file (`_index.md`) and a Vietnamese file (`_index.vi.md`),
matching the template's bilingual convention.

## How to use it in the template

1. Clone the template:
   ```bash
   git clone --recurse-submodules https://github.com/thienluhoan/fcj-workshop-template.git
   ```
2. Copy the `5-Workshop` folder from here into the template's `content/` directory,
   replacing the sample `5-Workshop`.
3. (Optional) Add screenshots under `static/images/5-Workshop/5.x-.../` and reference
   them in the Markdown with `![alt](/images/5-Workshop/...)`. See the "Screenshots to add"
   list below.
4. Preview locally:
   ```bash
   hugo server -D
   ```
   Then open http://localhost:1313. Use the language switcher to toggle VI/EN.

> Hugo (extended) is required by the theme. Install it from
> https://gohugo.io/installation/ if you do not have it.

## Screenshots to add (optional, recommended)

The text references AWS consoles and terminal output. To make the report richer, capture:

- 5.2: `aws sts get-caller-identity` output; IAM access-key creation screen.
- 5.4: Route 53 hosted zone with its 4 nameservers; registrar nameserver page; S3 bucket.
- 5.5: ACM certificate going from Pending validation to Issued.
- 5.6: CloudFront distribution details (status Deployed); S3 bucket policy.
- 5.7: Route 53 alias records; browser showing `https://pvpwork.com` with the padlock.

## Note on the content

The steps and values (domain `pvpwork.com`, bucket `pvpwork-fcaj-landing`,
distribution `ERM2MH804SI8I`, etc.) reflect a real deployment. Replace them with your
own identifiers when adapting this report, and do not copy it verbatim for submission.
