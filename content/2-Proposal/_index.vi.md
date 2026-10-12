---
title: "Đề xuất dự án"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

{{% notice warning %}}
⚠️ **Lưu ý:** Thông tin bên dưới chỉ mang tính tham khảo. Vui lòng **không sao chép nguyên văn** vào báo cáo của bạn, bao gồm cả cảnh báo này.
{{% /notice %}}

Trong phần này, tôi tóm tắt dự án dự định thực hiện trong chương trình.

# Landing Page FCAJ trên AWS
## Website tĩnh serverless, trả tiền theo mức dùng, với tên miền riêng

### 1. Tóm tắt
Dự án xuất bản landing page First Cloud AI Journey (FCAJ) ra Internet công khai trên tên miền riêng `pvpwork.com`, sử dụng kiến trúc AWS serverless hoàn toàn. Trang song ngữ (Việt/Anh), có tài liệu chương trình dạng PDF để tải về, cần tải nhanh cho người dùng ở Việt Nam, đồng thời chi phí vận hành gần như bằng không với lượng truy cập thấp và không thường xuyên.

### 2. Phát biểu bài toán
**Vấn đề.** Landing page FCAJ mới chỉ tồn tại dưới dạng tệp HTML cục bộ. Chưa có URL công khai, chưa có HTTPS, chưa có tên miền riêng, và chưa có cơ chế phân phối nhanh cho người dùng. Chạy một web server truyền thống cho một trang hoàn toàn tĩnh sẽ phát sinh chi phí và công bảo trì không cần thiết.

**Giải pháp.** Lưu các tệp ở chế độ riêng tư trong Amazon S3 và phục vụ toàn cầu qua Amazon CloudFront bằng HTTPS, bảo mật bằng chứng chỉ ACM miễn phí, với Amazon Route 53 trỏ tên miền riêng về CloudFront. S3 bucket luôn riêng tư; chỉ CloudFront đọc được qua Origin Access Control (OAC).

**Lợi ích.** Mặc định HTTPS, hiệu năng CDN toàn cầu, không có máy chủ phải vá lỗi, và chi phí hàng tháng gần bằng phí hosted zone của Route 53 (khoảng $0.50). Cùng thiết lập mở rộng được từ vài người tới lượng truy cập rất lớn mà không cần thay đổi.

### 3. Kiến trúc giải pháp
Website tĩnh phân phối qua CDN, với lưu trữ origin riêng tư:

+ **Amazon S3** — bucket riêng tư chứa `index.html`, hình ảnh và PDF.
+ **Amazon CloudFront** — CDN toàn cầu, kết thúc HTTPS, chuyển hướng HTTP→HTTPS, và function chuyển hướng `www` về apex.
+ **AWS Certificate Manager (ACM)** — chứng chỉ TLS miễn phí cho apex + www, ở `us-east-1`.
+ **Amazon Route 53** — hosted zone cho `pvpwork.com`, alias record trỏ về CloudFront.
+ **OAC + bucket policy** — giới hạn quyền đọc S3 chỉ cho một CloudFront distribution.

Chi tiết đầy đủ ở phần [Workshop](../5-Workshop/).

### 4. Triển khai kỹ thuật
**Các giai đoạn**
- Thiết kế kiến trúc và chọn region (S3 ở `ap-southeast-1`, ACM ở `us-east-1`).
- Tạo Route 53 hosted zone và chuyển nameserver từ nhà đăng ký (tắt DNSSEC trước).
- Yêu cầu và xác thực DNS cho chứng chỉ ACM.
- Tạo S3 bucket riêng tư và tải site lên.
- Tạo CloudFront với OAC, gắn bucket policy, thêm function www→apex.
- Trỏ DNS về CloudFront bằng alias record và kiểm tra end-to-end.

**Yêu cầu**: tài khoản AWS có quyền với S3/CloudFront/ACM/Route 53, AWS CLI v2, và một tên miền đã đăng ký.

### 5. Mốc thời gian
- Tuần 10: xây dựng toàn bộ phần triển khai trên AWS.
- Tuần 11: tên miền riêng, redirect www, kiểm tra, rà soát chi phí.
- Tuần 12: viết tài liệu và báo cáo workshop.

### 6. Ước tính chi phí
+ **Route 53 hosted zone:** ~$0.50/tháng.
+ **S3 lưu trữ + request:** vài cent/tháng cho ~1.5 MB tài nguyên.
+ **CloudFront:** nằm trong free tier với lưu lượng thấp; vài cent nếu vượt.
+ **Chứng chỉ ACM:** miễn phí.
+ **CloudFront Function (redirect www):** nằm trong free tier 2 triệu request/tháng.

**Tổng ước tính: khoảng $0.50–$1.00/tháng** với lượng truy cập thấp và không thường xuyên.

### 7. Đánh giá rủi ro
- **DNSSEC chặn đổi nameserver** — tác động trung bình, xử lý bằng cách tắt DNSSEC ở nhà đăng ký trước.
- **ACM kẹt ở trạng thái pending** — do nameserver chưa trỏ về Route 53; giảm thiểu bằng cách kiểm tra lan truyền trước khi kỳ vọng cấp chứng chỉ.
- **Vô tình để lộ S3 công khai** — giảm thiểu bằng Block Public Access cùng bucket policy giới hạn theo OAC.

### 8. Kết quả mong đợi
Một landing page hoạt động trực tuyến, bảo mật HTTPS tại `https://pvpwork.com`, phục vụ toàn cầu qua CloudFront, với tên miền phụ `www` chuyển hướng về apex. Một mô hình triển khai tái sử dụng được và một script `deploy.sh` một lệnh cho các lần cập nhật sau.
