---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

{{% notice warning %}}
⚠️ **Lưu ý:** Thông tin bên dưới chỉ mang tính tham khảo. Vui lòng **không sao chép nguyên văn** vào báo cáo của bạn, bao gồm cả cảnh báo này.
{{% /notice %}}

# Triển khai Landing Page tĩnh lên AWS với S3, CloudFront, Route 53 và ACM

#### Tổng quan

Trong workshop này, bạn sẽ đưa một website tĩnh (**landing page FCAJ**) lên Internet với tên miền riêng của mình, sử dụng kiến trúc AWS serverless hoàn toàn, trả tiền theo mức dùng. Không có web server nào phải quản lý và không phát sinh chi phí cố định hàng tháng ngoài vùng DNS (hosted zone).

Bạn sẽ lưu các tệp của website ở chế độ riêng tư trong **Amazon S3**, phân phối ra toàn cầu qua mạng CDN **Amazon CloudFront** bằng HTTPS, bảo mật tên miền bằng chứng chỉ miễn phí của **AWS Certificate Manager (ACM)**, và trỏ tên miền riêng về distribution bằng **Amazon Route 53**. S3 bucket luôn ở chế độ riêng tư: chỉ CloudFront đọc được, thông qua **Origin Access Control (OAC)**.

Đây là cách AWS khuyến nghị để host một trang tĩnh: rẻ, nhanh trên phạm vi toàn cầu, mặc định có HTTPS, và có thể mở rộng từ không tới lượng truy cập rất lớn mà không cần thay đổi gì.

#### Nội dung

1. [Tổng quan](5.1-Overview/)
2. [Điều kiện tiên quyết](5.2-Prerequisite/)
3. [Kiến trúc hệ thống](5.3-Architecture/)
4. [Tạo và tải lên S3](5.4-S3-Setup/)
5. [Yêu cầu chứng chỉ ACM](5.5-ACM-Certificate/)
6. [Tạo CloudFront distribution](5.6-CloudFront/)
7. [Cấu hình DNS Route 53](5.7-Route53-DNS/)
8. [Dọn dẹp tài nguyên](5.8-Cleanup/)
