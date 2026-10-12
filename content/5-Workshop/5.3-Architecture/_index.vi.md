---
title : "Kiến trúc hệ thống"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Kiến trúc hệ thống

Sơ đồ dưới đây mô tả luồng xử lý một yêu cầu và cách các dịch vụ AWS kết nối với nhau.

```
                                      (1) Phân giải DNS
          +-----------+   pvpwork.com  +--------------+
          | Trình duyệt| ------------> |  Route 53    |
          | người dùng | <------------ |  hosted zone |
          +-----------+   alias ->     +--------------+
                |             CloudFront
                | (2) Yêu cầu HTTPS (chứng chỉ TLS từ ACM)
                v
          +---------------------------+        +--------------------+
          |      Amazon CloudFront    | dùng   | Chứng chỉ ACM      |
          |  (CDN, các edge location) |------->| (us-east-1)        |
          |  - chuyển HTTP -> HTTPS   |        | pvpwork.com + www  |
          |  - www -> apex (function) |        +--------------------+
          +---------------------------+
                | (3) cache miss: lấy từ origin
                |     ký bằng OAC (SigV4)
                v
          +---------------------------+
          |        Amazon S3          |  (bucket riêng tư, không công khai)
          |  pvpwork-fcaj-landing     |
          |  index.html, assets/, docs/
          +---------------------------+
                ^
                | bucket policy: chỉ cho phép s3:GetObject
                | từ đúng CloudFront distribution này
```

#### Luồng xử lý một yêu cầu

1. **Phân giải DNS** — trình duyệt của người dùng tra cứu `pvpwork.com`. Route 53 trả về một **alias record** trỏ thẳng tới CloudFront distribution (cả A/IPv4 và AAAA/IPv6).
2. **Phân phối qua HTTPS** — trình duyệt kết nối tới edge location CloudFront gần nhất qua HTTPS. CloudFront xuất trình chứng chỉ ACM cho `pvpwork.com`. Mọi yêu cầu HTTP thường đều nhận lại mã 301 chuyển hướng sang HTTPS. Một **function viewer-request** của CloudFront chuyển hướng `www.pvpwork.com` về apex `pvpwork.com`.
3. **Lấy từ origin** — nếu tệp được yêu cầu chưa có trong cache tại edge, CloudFront lấy nó từ origin S3. Yêu cầu được ký bằng **Origin Access Control (OAC)** theo chuẩn SigV4, nên S3 có thể xác minh nó đến từ đúng distribution này.
4. **Lưu trữ riêng tư** — S3 bucket chặn mọi truy cập công khai. Bucket policy chỉ cho phép `s3:GetObject` **khi và chỉ khi** yêu cầu đến từ ARN của CloudFront distribution này. Không ai truy cập được tệp qua URL S3 trực tiếp.

#### Trách nhiệm của từng thành phần

| Thành phần | Trách nhiệm |
| --- | --- |
| **Route 53** | DNS có thẩm quyền cho tên miền; alias record trỏ về CloudFront |
| **CloudFront** | CDN toàn cầu, kết thúc HTTPS, chuyển hướng HTTP→HTTPS, chuyển hướng www→apex, cache |
| **ACM** | Chứng chỉ TLS miễn phí cho apex + www (ở `us-east-1`) |
| **S3** | Lưu trữ origin riêng tư cho các tệp website |
| **OAC + bucket policy** | Giới hạn quyền đọc S3 chỉ cho distribution CloudFront này |

#### Lưu ý về bảo mật

+ S3 bucket bật **Block Public Access**. Không có URL bucket công khai và không có object ACL cấp quyền đọc công khai.
+ CloudFront truy cập S3 qua **OAC** (bản thay thế hiện đại cho Origin Access Identity), được giới hạn bằng ARN distribution trong điều kiện `AWS:SourceArn` của bucket policy.
+ Chính sách giao thức phía người xem là **redirect-to-HTTPS**, và phiên bản TLS tối thiểu là **TLSv1.2_2021**.
