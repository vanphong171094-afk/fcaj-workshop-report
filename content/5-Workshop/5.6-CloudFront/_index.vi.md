---
title : "Tạo CloudFront distribution"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

Ở bước này, bạn tạo một Origin Access Control (OAC), tạo CloudFront distribution, rồi cập nhật bucket policy để chỉ distribution này đọc được S3.

#### Bước 1 — Tạo Origin Access Control (OAC)

OAC cho phép CloudFront ký các yêu cầu tới S3 bằng SigV4, nhờ đó bucket có thể ở chế độ riêng tư hoàn toàn.

```bash
aws cloudfront create-origin-access-control \
  --origin-access-control-config \
  Name=pvpwork-fcaj-oac,Description="OAC for pvpwork fcaj landing",SigningProtocol=sigv4,SigningBehavior=always,OriginAccessControlOriginType=s3
```

Ghi lại **OAC Id** trong kết quả (ví dụ `E2HFW5VE5I8YJO`).

#### Bước 2 — Tạo CloudFront distribution

Chuẩn bị một tệp cấu hình `cf-config.json`. Các điểm quan trọng:

+ `Origins` trỏ về domain của bucket: `pvpwork-fcaj-landing.s3.ap-southeast-1.amazonaws.com`, kèm `OriginAccessControlId` ở trên.
+ `Aliases` liệt kê `pvpwork.com` và `www.pvpwork.com`.
+ `DefaultRootObject` là `index.html`.
+ `ViewerProtocolPolicy` là `redirect-to-https`.
+ `ViewerCertificate` dùng ARN chứng chỉ ACM, `SSLSupportMethod=sni-only`, `MinimumProtocolVersion=TLSv1.2_2021`.
+ `CachePolicyId` là `658327ea-f89d-4fab-a63d-7e88639e58f6` (managed policy "CachingOptimized" của AWS).
+ `PriceClass` là `PriceClass_200` (gồm các edge location ở châu Á, bỏ các khu vực đắt đỏ hơn để tiết kiệm).

```bash
aws cloudfront create-distribution --distribution-config file://cf-config.json
```

Kết quả trả về **Distribution Id** (ví dụ `ERM2MH804SI8I`) và **tên miền CloudFront** (ví dụ `d3pj405a0bg80k.cloudfront.net`). Trạng thái ban đầu là `InProgress`; CloudFront cần vài phút để triển khai ra toàn bộ edge.

{{% notice note %}}
Chứng chỉ ACM phải ở trạng thái `ISSUED` tại thời điểm tạo distribution. Nếu chưa, hãy hoàn tất bước 5.5 trước.
{{% /notice %}}

#### Bước 3 — Gắn bucket policy cho phép OAC

Giờ cho phép CloudFront đọc bucket. Tạo `bucket-policy.json`, chỉ cho phép đúng distribution ARN:

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

#### Bước 4 — Chờ distribution triển khai xong

```bash
aws cloudfront get-distribution --id ERM2MH804SI8I --query 'Distribution.Status'
```

Chờ tới khi trạng thái là `Deployed`. Sau đó bạn có thể test bằng tên miền CloudFront trước khi DNS trỏ về:

```bash
curl -I https://d3pj405a0bg80k.cloudfront.net/
```

Lệnh này phải trả về `HTTP/2 200`.
