---
title : "Cấu hình DNS Route 53"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

Ở bước này, bạn trỏ tên miền về CloudFront bằng alias record, thêm chuyển hướng `www` → apex, rồi kiểm tra toàn bộ hệ thống.

#### Bước 1 — Tạo alias record cho apex và www

CloudFront có một hosted zone Id cố định trên toàn cầu là `Z2FDTNDATAQYW2`. Dùng nó trong các bản ghi alias. Tạo `dns-alias.json` với bốn bản ghi: A và AAAA cho cả apex và www, tất cả trỏ về tên miền CloudFront.

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

Lặp lại khối trên cho `www.pvpwork.com.` và cho loại `AAAA`, rồi áp dụng:

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id Z0036429187B89PBOPZ1R \
  --change-batch file://dns-alias.json
```

{{% notice tip %}}
Alias record của Route 53 giải quyết được **apex domain** (`pvpwork.com`) trỏ thẳng về CloudFront — điều mà bản ghi CNAME thường không làm được ở tên miền gốc. Đây là lý do chính nên dùng Route 53 cho zone.
{{% /notice %}}

#### Bước 2 — Chuyển hướng www → apex (miễn phí)

Để `pvpwork.com` là URL chính duy nhất (tốt cho SEO), chuyển hướng `www` về apex bằng một **CloudFront Function** chạy ở sự kiện viewer-request. Function nằm trong free tier rộng rãi (2 triệu lần gọi/tháng), nên không phát sinh chi phí thực tế.

Mã function (`redirect-www.js`):

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

Tạo, publish, rồi gắn vào distribution:

```bash
# Tạo function
aws cloudfront create-function \
  --name pvpwork-www-redirect \
  --function-config Comment="Redirect www to apex",Runtime="cloudfront-js-2.0" \
  --function-code fileb://redirect-www.js

# Publish lên stage LIVE (dùng ETag từ kết quả create)
aws cloudfront publish-function --name pvpwork-www-redirect --if-match <ETAG>
```

Sau đó gắn function vào default cache behavior của distribution (sự kiện `viewer-request`) bằng `update-distribution`, và chờ trạng thái trở lại `Deployed`.

#### Bước 3 — Kiểm tra toàn bộ hệ thống

```bash
# DNS phân giải về IP của CloudFront
dig +short A pvpwork.com @8.8.8.8

# Apex phục vụ qua HTTPS
curl -sS -o /dev/null -w "HTTP %{http_code}\n" https://pvpwork.com/

# HTTP phải chuyển hướng sang HTTPS
curl -sS -o /dev/null -w "HTTP %{http_code} -> %{redirect_url}\n" http://pvpwork.com/

# www phải 301 về apex, giữ nguyên path
curl -sS -o /dev/null -w "HTTP %{http_code} -> %{redirect_url}\n" https://www.pvpwork.com/

# PDF trả về đúng content-type
curl -sS -o /dev/null -w "HTTP %{http_code}  type=%{content_type}\n" https://pvpwork.com/docs/fcaj-program-vi.pdf
```

Kết quả mong đợi:

| Kiểm tra | Kết quả |
| --- | --- |
| `https://pvpwork.com/` | HTTP 200 |
| `https://www.pvpwork.com/` | HTTP 301 → `https://pvpwork.com/` |
| `http://pvpwork.com/` | HTTP 301 → HTTPS |
| PDF | HTTP 200, `application/pdf` |

{{% notice note %}}
Một bản ghi apex mới tạo có thể cần vài phút để lan truyền tới mọi DNS resolver. Nếu một resolver tạm thời chưa thấy bản ghi apex trong khi `www` đã thấy, hãy chờ vài phút rồi kiểm tra lại.
{{% /notice %}}

Khi tất cả kiểm tra đều đạt, website đã hoạt động trực tuyến tại `https://pvpwork.com`.
