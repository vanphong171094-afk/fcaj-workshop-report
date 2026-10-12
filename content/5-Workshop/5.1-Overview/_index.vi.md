---
title : "Tổng quan"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Bạn sẽ xây dựng gì

Bạn sẽ triển khai một website tĩnh lên AWS và cho phép truy cập qua tên miền riêng bằng HTTPS. Khi kết thúc workshop, truy cập `https://ten-mien-cua-ban.com` sẽ trả về landing page từ edge location CloudFront gần nhất, và tên miền phụ `www` sẽ chuyển hướng về tên miền gốc (apex).

Trong báo cáo này, các giá trị thực tế được sử dụng là:

| Thành phần | Giá trị |
| --- | --- |
| Tên miền | `pvpwork.com` (apex) và `www.pvpwork.com` |
| Region (S3) | `ap-southeast-1` (Singapore) |
| Region (ACM) | `us-east-1` (bắt buộc cho CloudFront) |

#### Vì sao chọn kiến trúc này

Một landing page tĩnh không có mã chạy phía máy chủ, nên chạy một web server truyền thống sẽ rất lãng phí. Mô hình S3 + CloudFront mang lại:

+ **Chi phí thấp** — bạn chỉ trả cho dung lượng lưu trữ và lượng truy cập thực tế. Với lượng truy cập thấp và không thường xuyên, hóa đơn gần bằng chi phí hosted zone của Route 53 (khoảng **$0.50/tháng**).
+ **Hiệu năng toàn cầu** — CloudFront cache các tệp của bạn tại các edge location trên khắp thế giới, bao gồm châu Á, nên người dùng ở Việt Nam được phục vụ nhanh.
+ **Mặc định có HTTPS** — một chứng chỉ ACM miễn phí bảo mật tên miền, và mọi yêu cầu HTTP đều được chuyển hướng sang HTTPS.
+ **Bảo mật** — S3 bucket không bao giờ công khai. Chỉ CloudFront đọc được, được đảm bảo bằng Origin Access Control (OAC) và bucket policy.
+ **Khả năng mở rộng** — cùng một thiết lập phục vụ được mười người truy cập hay mười triệu người mà không cần thay đổi gì.

#### Các dịch vụ AWS chính được sử dụng

+ **Amazon S3** — lưu trữ đối tượng, chứa các tệp của website (HTML, hình ảnh, PDF). Cấu hình ở chế độ bucket riêng tư.
+ **Amazon CloudFront** — mạng phân phối nội dung (CDN), cache và phục vụ các tệp trên toàn cầu qua HTTPS, và là thành phần duy nhất được phép đọc bucket.
+ **AWS Certificate Manager (ACM)** — cấp và tự động gia hạn chứng chỉ TLS/SSL miễn phí cho tên miền. Phải nằm ở `us-east-1` để CloudFront sử dụng được.
+ **Amazon Route 53** — dịch vụ DNS. Chứa hosted zone của tên miền và dùng alias record để trỏ tên miền về CloudFront distribution.
