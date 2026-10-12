---
title : "Điều kiện tiên quyết"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

Trước khi bắt đầu, hãy đảm bảo bạn có những điều sau.

#### 1. Tài khoản AWS với quyền phù hợp

Bạn cần một IAM user hoặc role được phép làm việc với S3, CloudFront, ACM và Route 53. Với tài khoản lab cá nhân, policy quản lý **`AdministratorAccess`** là đơn giản nhất. Trong môi trường production, hãy thu hẹp quyền chỉ còn các dịch vụ này.

{{% notice tip %}}
Kiểm tra bạn đang xác thực bằng danh tính nào trước khi bắt đầu:
```bash
aws sts get-caller-identity
```
Lệnh này trả về `Account`, `UserId` và `Arn` của bạn. Nếu báo lỗi, nghĩa là credentials chưa được cấu hình.
{{% /notice %}}

#### 2. Đã cài và cấu hình AWS CLI v2

Cài AWS CLI v2, sau đó cấu hình:

```bash
aws configure
```

Nhập Access Key ID, Secret Access Key, region mặc định (ví dụ `ap-southeast-1`), và định dạng đầu ra `json`.

{{% notice info %}}
Bạn có thể tạo Access Key trong AWS Console: **IAM → Users → user của bạn → Security credentials → Create access key → Command Line Interface (CLI)**.
{{% /notice %}}

#### 3. Một tên miền đã đăng ký

Bạn cần một tên miền do mình quản lý, ví dụ `pvpwork.com`. Tên miền có thể đăng ký ở bất kỳ đâu (Route 53, hoặc nhà cung cấp bên thứ ba như Mắt Bão, GoDaddy, Namecheap). Trong workshop này, tên miền được đăng ký ở nhà cung cấp bên thứ ba, và chúng ta sẽ chuyển việc quản lý DNS sang Route 53.

{{% notice warning %}}
Nếu tên miền của bạn đang **bật DNSSEC** tại nhà đăng ký, bạn phải **tắt DNSSEC trước**. DNSSEC ký các bản ghi DNS bằng khóa gắn với nameserver hiện tại; đổi nameserver khi DNSSEC còn bật sẽ phá vỡ chuỗi tin cậy, và hầu hết nhà đăng ký sẽ chặn việc đổi nameserver cho tới khi DNSSEC được tắt.
{{% /notice %}}

#### 4. Các tệp của website

Một thư mục chứa các tệp tĩnh cần xuất bản. Trong workshop này là landing page FCAJ:

```
fcaj-landing/
├── index.html
├── assets/        (logo, hình ảnh, favicon — .webp / .svg)
└── docs/          (tài liệu chương trình dạng PDF)
```

Không cần bước build và không cần cơ sở dữ liệu — trang web hoàn toàn là tệp tĩnh.
