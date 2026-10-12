---
title : "Yêu cầu chứng chỉ ACM"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

Ở bước này, bạn yêu cầu một chứng chỉ TLS/SSL miễn phí cho tên miền và xác thực nó bằng DNS.

{{% notice warning %}}
Chứng chỉ dùng cho CloudFront **bắt buộc phải nằm ở region `us-east-1`** (N. Virginia), bất kể S3 bucket của bạn ở region nào. Đây là lỗi phổ biến nhất khi làm workshop này.
{{% /notice %}}

#### Bước 1 — Yêu cầu chứng chỉ

Yêu cầu một chứng chỉ cho cả tên miền apex và tên miền phụ `www`, xác thực bằng phương thức DNS.

```bash
aws acm request-certificate \
  --region us-east-1 \
  --domain-name pvpwork.com \
  --subject-alternative-names www.pvpwork.com \
  --validation-method DNS \
  --key-algorithm RSA_2048 \
  --tags Key=Project,Value=fcaj-landing
```

Kết quả trả về **ARN của chứng chỉ**, ví dụ:
`arn:aws:acm:us-east-1:669167565740:certificate/e7fd36b9-...`. Ghi lại ARN này.

#### Bước 2 — Lấy các bản ghi xác thực DNS

ACM sinh ra một bản ghi CNAME cho mỗi tên miền. Lấy chúng ra:

```bash
aws acm describe-certificate \
  --region us-east-1 \
  --certificate-arn <ARN-CHUNG-CHI> \
  --query 'Certificate.DomainValidationOptions[].ResourceRecord'
```

#### Bước 3 — Thêm bản ghi xác thực vào Route 53

Tạo một tệp change batch (`acm-validation.json`) với các cặp Name/Value lấy từ bước trên:

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

Áp dụng vào hosted zone của bạn:

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id Z0036429187B89PBOPZ1R \
  --change-batch file://acm-validation.json
```

#### Bước 4 — Chờ chứng chỉ được cấp

ACM tự động xác thực khi tìm thấy các bản ghi DNS. Kiểm tra trạng thái:

```bash
aws acm describe-certificate \
  --region us-east-1 \
  --certificate-arn <ARN-CHUNG-CHI> \
  --query 'Certificate.Status'
```

Trạng thái chuyển từ `PENDING_VALIDATION` sang `ISSUED`, thường mất vài phút.

{{% notice warning %}}
Nếu trạng thái **kẹt mãi ở `PENDING_VALIDATION`**, gần như chắc chắn do nameserver của tên miền **chưa trỏ về Route 53**. ACM xác thực bằng cách phân giải các bản ghi qua DNS công khai; nếu nhà đăng ký vẫn giữ nameserver cũ, ACM không bao giờ thấy bản ghi xác thực dù chúng đã nằm đúng trong Route 53. Hãy hoàn tất việc đổi nameserver ở bước 5.4 trước.
{{% /notice %}}
