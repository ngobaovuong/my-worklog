---
title: "Worklog Tuần 2"
date: 2026-08-10
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:
* Nghiên cứu chuyên sâu và thực hành nhóm dịch vụ cốt lõi: AWS IAM, Amazon EC2 và Amazon S3.
* Quản lý phân quyền danh tính theo nguyên tắc đặc quyền tối thiểu (Least Privilege) qua IAM Users, Groups, Roles và Policies.
* Khởi tạo, cấu hình mạng, thiết lập Security Group và quản trị từ xa máy chủ ảo Amazon EC2.
* Lưu trữ đối tượng với Amazon S3, quản lý vòng đời dữ liệu, phân quyền bảo mật Bucket Policy và cấu hình hosting website tĩnh.
* Kết hợp IAM Instance Profile với EC2 để truy cập S3 mà không lưu trữ cứng thông tin xác thực (Access Keys).

### Các công việc triển khai trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Nghiên cứu cấu trúc tài liệu IAM Policy dạng JSON (Version, Statement, Effect, Action, Resource, Condition).<br>- Phân biệt Identity-based Policy, Resource-based Policy và Permission Boundary.<br>- Tạo IAM Group `Developers`, gắn các Policy tùy biến có giới hạn dịch vụ và kiểm tra phân quyền người dùng. | 10/08/2026 | 10/08/2026 | https://docs.aws.amazon.com/IAM/latest/UserGuide/ |
| 3 | - Tìm hiểu cơ chế ảo hóa Nitro System của AWS, các nhóm instance EC2 (General Purpose, Compute, Memory, Storage Optimized).<br>- Tạo Key Pair (loại RSA, format `.pem` và `.ppk`), phân quyền tệp `chmod 400 key.pem`.<br>- Khởi tạo một máy ảo EC2 chạy Amazon Linux 2023 (`t3.micro` thuộc Free Tier) tại Availability Zone `ap-southeast-1a`. | 11/08/2026 | 11/08/2026 | https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ |
| 4 | - Cấu hình Security Group (Inbound rules: mở port 22 cho SSH từ My IP, port 80 cho HTTP từ 0.0.0.0/0).<br>- Kết nối vào máy ảo qua SSH terminal: `ssh -i key.pem ec2-user@<public-ip>`.<br>- Cập nhật hệ thống bằng `dnf update -y`, cài đặt Apache Web Server (`httpd`), tạo trang `index.html` tùy biến và kiểm tra truy cập HTTP qua trình duyệt. | 12/08/2026 | 12/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| 5 | - Nghiên cứu dịch vụ lưu trữ đối tượng Amazon S3: khái niệm Bucket, Object Key, Metadata, Storage Classes (Standard, Infrequent Access, Glacier).<br>- Tạo S3 Bucket với tên duy nhất toàn cầu (Globally Unique Name), bật tính năng Bucket Versioning.<br>- Sử dụng AWS CLI để đồng bộ và thao tác tệp: `aws s3 cp`, `aws s3 sync`, `aws s3 ls`. | 13/08/2026 | 13/08/2026 | https://docs.aws.amazon.com/AmazonS3/latest/userguide/ |
| 6 | - Tắt cấu hình `Block Public Access` của S3, soạn thảo Bucket Policy cấp quyền `s3:GetObject` công khai cho anonymous user.<br>- Kích hoạt tính năng Static Website Hosting và truy cập web qua S3 Endpoint URL.<br>- Tạo IAM Role với quyền `AmazonS3ReadOnlyAccess`, gán Instance Profile vào EC2 và kiểm tra câu lệnh `aws s3 ls` trực tiếp từ trong máy chủ mà không cần config access keys. | 14/08/2026 | 14/08/2026 | https://cloudjourney.awsstudygroup.com/ |

### Kết quả đạt được tuần 2:
* **Mức độ hoàn thành:** 100%.
* **Kiến thức lý thuyết:**
  * Hiểu cơ chế phân quyền RBAC và ABAC trong IAM, sự khác biệt giữa IAM User (người dùng lâu dài) và IAM Role (danh tính tạm thời dùng AWS STS token).
  * Hiểu vòng đời của EC2 Instance (Pending, Running, Stopping, Stopped, Terminated) và cơ chế định giá (On-Demand, Spot, Reserved Instances, Savings Plans).
  * Nắm vững cơ chế tính nhất quán của S3 (Strong Read-After-Write Consistency).
* **Kỹ năng thực hành:**
  * Xây dựng hoàn chỉnh một Web Server chạy trên EC2 được bảo vệ bởi Security Group đúng chuẩn.
  * Xuất bản website tĩnh hoàn toàn không cần server trên Amazon S3.
  * Loại bỏ rủi ro rò rỉ credential bằng cách sử dụng IAM Role gán vào EC2 Instance Profile.