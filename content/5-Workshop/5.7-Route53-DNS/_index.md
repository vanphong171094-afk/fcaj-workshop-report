---
title : "Configure Route 53 DNS"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

In this step you point the domain at CloudFront with alias records, add a `www` → apex redirect, then verify the whole system.

#### Step 1 — Create alias records for apex and www

CloudFront has a fixed global hosted zone Id of `Z2FDTNDATAQYW2`. Use it in the alias records. Create `dns-alias.json` with four records: A and AAAA for both apex and www, all targeting the CloudFront domain.

```json
{
  "Comment": "Alias apex and www to CloudFront",
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "pvpwork.com.",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "Z2FDTNDATAQYW2",
          "DNSName": "d3pj405a0bg80k.cloudfront.net.",
          "EvaluateTargetHealth": false
        }
      }
    }
  ]
}
```

Repeat the block above for `www.pvpwork.com.` and for type `AAAA`, then apply:

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id Z0036429187B89PBOPZ1R \
  --change-batch file://dns-alias.json
```

{{% notice tip %}}
A Route 53 alias record can point an **apex domain** (`pvpwork.com`) straight at CloudFront — something a plain CNAME cannot do at the zone root. This is the main reason to use Route 53 for the zone.
{{% /notice %}}

#### Step 2 — Redirect www → apex (free)

To make `pvpwork.com` the single canonical URL (good for SEO), redirect `www` to the apex with a **CloudFront Function** running on the viewer-request event. The function sits inside a generous free tier (2 million invocations/month), so there is no real cost.

Function code (`redirect-www.js`):

```javascript
function handler(event) {
    var request = event.request;
    var host = request.headers.host.value;
    if (host === 'www.pvpwork.com') {
        return {
            statusCode: 301,
            statusDescription: 'Moved Permanently',
            headers: {
                location: { value: 'https://pvpwork.com' + request.uri }
            }
        };
    }
    return request;
}
```

Create, publish, then attach it to the distribution:

```bash
# Create the function
aws cloudfront create-function \
  --name pvpwork-www-redirect \
  --function-config Comment="Redirect www to apex",Runtime="cloudfront-js-2.0" \
  --function-code fileb://redirect-www.js

# Publish to the LIVE stage (use the ETag from the create response)
aws cloudfront publish-function --name pvpwork-www-redirect --if-match <ETAG>
```

Then attach the function to the distribution's default cache behavior (`viewer-request` event) with `update-distribution`, and wait for the status to return to `Deployed`.

#### Step 3 — Verify the whole system

```bash
# DNS resolves to CloudFront IPs
dig +short A pvpwork.com @8.8.8.8

# Apex serves over HTTPS
curl -sS -o /dev/null -w "HTTP %{http_code}\n" https://pvpwork.com/

# HTTP must redirect to HTTPS
curl -sS -o /dev/null -w "HTTP %{http_code} -> %{redirect_url}\n" http://pvpwork.com/

# www must 301 to apex, preserving the path
curl -sS -o /dev/null -w "HTTP %{http_code} -> %{redirect_url}\n" https://www.pvpwork.com/

# PDF returns the correct content-type
curl -sS -o /dev/null -w "HTTP %{http_code}  type=%{content_type}\n" https://pvpwork.com/docs/fcaj-program-vi.pdf
```

Expected results:

| Check | Result |
| --- | --- |
| `https://pvpwork.com/` | HTTP 200 |
| `https://www.pvpwork.com/` | HTTP 301 → `https://pvpwork.com/` |
| `http://pvpwork.com/` | HTTP 301 → HTTPS |
| PDF | HTTP 200, `application/pdf` |

{{% notice note %}}
A freshly created apex record may take a few minutes to propagate to every DNS resolver. If one resolver temporarily does not see the apex record while `www` already resolves, wait a few minutes and check again.
{{% /notice %}}

When all checks pass, the website is live at `https://pvpwork.com`.
