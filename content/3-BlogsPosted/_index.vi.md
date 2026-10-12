---
title: "Bài viết đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

{{% notice warning %}}
⚠️ **Lưu ý:** Thông tin bên dưới chỉ mang tính tham khảo. Vui lòng **không sao chép nguyên văn** vào báo cáo của bạn, bao gồm cả cảnh báo này.
{{% /notice %}}

Phần này liệt kê các bài viết tôi đã chia sẻ với cộng đồng [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj) trong chương trình. Mỗi bài ghi lại một điều tôi học được khi xây dựng dự án cuối khóa.

### [Bài 1 - Host website tĩnh riêng tư với S3 + CloudFront OAC](3.1-Blog1/)
Bài viết giải thích cách phục vụ một website tĩnh từ S3 bucket **riêng tư** bằng cách đặt CloudFront phía trước với Origin Access Control (OAC). Bài nói về lý do nên giữ Block Public Access, cách bucket policy giới hạn quyền theo một distribution ARN duy nhất, và vì sao OAC thay thế Origin Access Identity cũ.

### [Bài 2 - HTTPS miễn phí cho mọi tên miền với AWS Certificate Manager](3.2-Blog2/)
Bài hướng dẫn yêu cầu chứng chỉ TLS miễn phí trong ACM, xác thực bằng bản ghi DNS trong Route 53, và lỗi thường gặp khi chứng chỉ kẹt ở `PENDING_VALIDATION` vì nameserver của tên miền chưa trỏ về Route 53. Bài cũng lưu ý chứng chỉ cho CloudFront bắt buộc nằm ở `us-east-1`.

### [Bài 3 - Trỏ apex domain về CloudFront và chuyển hướng www](3.3-Blog3/)
Bài trình bày cách alias record của Route 53 cho phép apex domain trỏ thẳng về CloudFront (điều CNAME thường không làm được), và cách một CloudFront Function nhỏ chuyển hướng `www` về apex để có một URL chính duy nhất, tất cả đều trong free tier.
