---
title: "Worklog Tuần 3"
date: 2026-08-17
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:
* Nghiên cứu sâu về kiến trúc mạng cô lập Amazon Virtual Private Cloud (Amazon VPC).
* Nắm vững kỹ thuật phân chia dải mạng con CIDR (IPv4 Subnetting) và phân chia vùng mạng Public / Private Subnet.
* Thiết kế và cấu hình bảng định tuyến Route Tables, cổng kết nối Internet Gateway (IGW) và cơ chế dịch địa chỉ mạng NAT.
* Phân tích và cấu hình 2 tầng tường lửa: Security Groups (Stateful) và Network Access Control Lists (Stateless).

### Các công việc triển khai trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu lý thuyết mạng máy tính đám mây, cấu trúc khối địa chỉ CIDR theo chuẩn RFC 1918.<br>- Khởi tạo Custom VPC với dải IP `10.0.0.0/16` (cung cấp 65,536 địa chỉ IP).<br>- Cấu hình kích hoạt các thuộc tính DNS Resolution và DNS Hostnames cho VPC. | 17/08/2026 | 17/08/2026 | https://docs.aws.amazon.com/vpc/latest/userguide/ |
| 3 | - Phân tích cơ chế AWS dành riêng 5 địa chỉ IP trong mỗi Subnet (.0, .1, .2, .3, .255).<br>- Tạo Public Subnet A (`10.0.1.0/24`) tại AZ `ap-southeast-1a` và Public Subnet B (`10.0.2.0/24`) tại AZ `ap-southeast-1b`.<br>- Tạo Private Subnet A (`10.0.10.0/24`) tại AZ `ap-southeast-1a` và Private Subnet B (`10.0.20.0/24`) tại AZ `ap-southeast-1b`. | 18/08/2026 | 18/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| 4 | - Khởi tạo Internet Gateway (IGW) và gắn (attach) vào Custom VPC.<br>- Tạo Custom Route Table cho Public Subnet, thêm tuyến đường default route `0.0.0.0/0` trỏ đến Internet Gateway.<br>- Liên kết (associate) Public Subnet A và B vào Public Route Table.<br>- Kiểm tra Main Route Table mặc định (chỉ chứa route nội bộ `10.0.0.0/16 local`) dùng riêng cho Private Subnets. | 19/08/2026 | 19/08/2026 | https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html |
| 5 | - Triển khai một EC2 Instance vào Public Subnet A, bật tùy chọn Auto-assign Public IP.<br>- Triển khai một EC2 Instance thứ hai vào Private Subnet A (chỉ có Private IP).<br>- Thiết lập máy chủ Bastion Host (Jump Host) trên Public Subnet để kết nối SSH an toàn vào máy chủ Private Subnet.<br>- Tìm hiểu cơ chế hoạt động của NAT Gateway (hỗ trợ Private Subnet tải bản vá từ Internet). | 20/08/2026 | 20/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| 6 | - Cấu hình Network ACL (NACL) tùy biến: phân tích cơ chế đánh số quy tắc (Rule Numbers) từ nhỏ đến lớn và tính chất Stateless.<br>- So sánh chi tiết sự khác nhau trong thực tế giữa Security Group (áp dụng mức ENI/Instance) và Network ACL (áp dụng mức Subnet).<br>- Kiểm thử chặn IP cụ thể thông qua Inbound NACL Deny Rule và quan sát kết quả. | 21/08/2026 | 21/08/2026 | https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html |

### Kết quả đạt được tuần 3:
* **Mức độ hoàn thành:** 100%.
* **Kiến thức lý thuyết:**
  * Hiểu rõ kiến trúc Multi-AZ VPC chuẩn doanh nghiệp đảm bảo tính cô lập và dự phòng địa lý.
  * Phân biệt rành mạch cơ chế hoạt động của Internet Gateway (phi trạng thái, co giãn tự động theo băng thông của VPC) và NAT Gateway (dịch địa chỉ mạng cho tài nguyên private đi ra ngoài).
  * Nắm chắc bản chất Stateful (tự mở chiều về đối với Security Group) và Stateless (phải định nghĩa cả Inbound và Outbound đối với NACL).
* **Kỹ năng thực hành:**
  * Tự tay thiết kế và triển khai trọn vẹn mô hình VPC 2 lớp mạng (Public/Private Subnet) trên nhiều Vùng Khả Dụng.
  * Xây dựng mô hình pháo đài Bastion Host để quản trị hệ thống máy chủ nội bộ mà không để lộ cổng kết nối ra ngoài Internet.