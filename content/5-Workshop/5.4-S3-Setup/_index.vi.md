---
title : "Tạo và tải lên S3"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

Ở bước này, bạn tạo Route 53 hosted zone để quản lý DNS, sau đó tạo một S3 bucket riêng tư và tải các tệp website lên.

#### Bước 1 — Tạo Route 53 hosted zone

Hosted zone phải tồn tại trước, vì chứng chỉ ACM (bước kế tiếp) được xác thực bằng các bản ghi DNS đặt trong zone này.

```bash
aws route53 create-hosted-zone \
  --name pvpwork.com \
  --caller-reference "pvpwork-$(date +%s)" \
  --hosted-zone-config Comment="FCAJ landing page"
```

Kết quả trả về bao gồm **Id của hosted zone** (ví dụ `Z0036429187B89PBOPZ1R`) và bốn **NameServers**. Ghi lại cả hai.

{{% notice info %}}
Vào trang quản lý tên miền của nhà đăng ký và thay các nameserver hiện tại bằng bốn nameserver Route 53 trong kết quả trả về. Với tên miền `.com`, quá trình lan truyền thường mất vài phút tới vài giờ. Bước này làm thủ công, bên ngoài AWS.
{{% /notice %}}

Kiểm tra nameserver sau khi đã lan truyền:

```bash
dig +short NS pvpwork.com @8.8.8.8
```

Kết quả phải liệt kê bốn nameserver `awsdns`, không còn nameserver cũ của nhà đăng ký.

#### Bước 2 — Tạo S3 bucket riêng tư

Tạo bucket ở region Singapore (`ap-southeast-1`), gần người dùng ở Việt Nam. Bucket mới mặc định đã bật **Block Public Access** — hãy giữ nguyên như vậy.

```bash
aws s3api create-bucket \
  --bucket pvpwork-fcaj-landing \
  --region ap-southeast-1 \
  --create-bucket-configuration LocationConstraint=ap-southeast-1
```

{{% notice note %}}
Tên S3 bucket là duy nhất trên toàn cầu. Nếu `pvpwork-fcaj-landing` đã có người dùng, hãy chọn tên khác và dùng nhất quán tên đó ở các bước sau.
{{% /notice %}}

#### Bước 3 — Tải các tệp website lên

Dùng `aws s3 sync` để tải toàn bộ thư mục. Loại bỏ các tệp rác cục bộ và những gì bạn không muốn xuất bản.

```bash
aws s3 sync /duong-dan/toi/fcaj-landing s3://pvpwork-fcaj-landing \
  --exclude ".DS_Store" \
  --exclude "*/.DS_Store" \
  --exclude "README.txt" \
  --delete
```

+ `--delete` xóa khỏi bucket những đối tượng không còn tồn tại ở cục bộ, giữ cho bucket luôn đồng bộ.
+ Lần tải này bao gồm `index.html`, toàn bộ thư mục `assets/`, và các tệp PDF trong `docs/`.

Xác nhận các tệp đã có:

```bash
aws s3 ls s3://pvpwork-fcaj-landing --recursive
```

{{% notice tip %}}
Bucket đang ở chế độ riêng tư, nên bạn **chưa thể** mở các tệp này từ URL S3 trên trình duyệt. Chúng chỉ truy cập được qua CloudFront, thiết lập ở bước 5.6.
{{% /notice %}}
