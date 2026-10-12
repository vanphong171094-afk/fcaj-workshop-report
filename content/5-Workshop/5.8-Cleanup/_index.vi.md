---
title : "Dọn dẹp tài nguyên"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

Để tránh phát sinh chi phí, hãy xóa các tài nguyên bạn đã tạo. Xóa theo thứ tự dưới đây, vì một số tài nguyên phụ thuộc vào tài nguyên khác.

{{% notice warning %}}
Chỉ dọn dẹp nếu bạn muốn gỡ bỏ dự án. Nếu đây là trang web đang chạy mà bạn muốn giữ lại, **hãy bỏ qua trang này**. Chi phí vận hành rất thấp (khoảng $0.50/tháng cho hosted zone cộng vài cent tiền sử dụng).
{{% /notice %}}

#### Bước 1 — Vô hiệu hóa và xóa CloudFront distribution

Một distribution phải được vô hiệu hóa (disabled) và triển khai xong hoàn toàn trước khi có thể xóa.

```bash
# Vô hiệu hóa: lấy config, đặt Enabled=false, rồi update-distribution kèm ETag.
# Chờ tới khi trạng thái là Deployed trở lại, sau đó xóa:
aws cloudfront delete-distribution --id ERM2MH804SI8I --if-match <ETAG>
```

#### Bước 2 — Xóa CloudFront Function và OAC

```bash
aws cloudfront delete-function --name pvpwork-www-redirect --if-match <ETAG>
aws cloudfront delete-origin-access-control --id E2HFW5VE5I8YJO --if-match <ETAG>
```

#### Bước 3 — Làm rỗng và xóa S3 bucket

```bash
aws s3 rm s3://pvpwork-fcaj-landing --recursive
aws s3api delete-bucket --bucket pvpwork-fcaj-landing --region ap-southeast-1
```

#### Bước 4 — Xóa chứng chỉ ACM

```bash
aws acm delete-certificate \
  --region us-east-1 \
  --certificate-arn arn:aws:acm:us-east-1:669167565740:certificate/e7fd36b9-...
```

#### Bước 5 — Xóa các bản ghi Route 53 và hosted zone

Trước tiên xóa các bản ghi alias và bản ghi xác thực bạn đã thêm (một hosted zone chỉ xóa được khi chỉ còn các bản ghi NS và SOA mặc định), rồi xóa zone:

```bash
# Xóa các record set alias (A/AAAA) và xác thực ACM (CNAME) bằng
# change-resource-record-sets với Action=DELETE, sau đó:
aws route53 delete-hosted-zone --id Z0036429187B89PBOPZ1R
```

{{% notice tip %}}
Nếu bạn giữ lại tên miền, có thể để nguyên hosted zone và chỉ xóa CloudFront, function, OAC, S3 và chứng chỉ. Xóa hosted zone đồng nghĩa với việc sau này phải tạo lại và đổi nameserver một lần nữa.
{{% /notice %}}

#### Kiểm tra không còn gì sót lại

```bash
aws s3 ls | grep pvpwork-fcaj-landing            # không trả về gì
aws cloudfront list-distributions                 # distribution đã mất
aws route53 list-hosted-zones-by-name --dns-name pvpwork.com
```
