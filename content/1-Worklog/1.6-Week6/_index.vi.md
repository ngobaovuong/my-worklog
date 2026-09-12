---
title: "Worklog Tuần 6"
date: 2026-09-07
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:
* Nghiên cứu và xây dựng kiến trúc ứng dụng web có tính sẵn sàng cao (High Availability) và khả năng chịu lỗi (Fault Tolerance).
* Nắm vững cơ chế phân phối tải lưu lượng mạng bằng Elastic Load Balancing (ELB) - trọng tâm là Application Load Balancer (ALB).
* Tìm hiểu cơ chế tự động co giãn tài nguyên theo tải với Auto Scaling Groups (ASG).
* Kết hợp ALB và ASG qua nhiều Availability Zones để tạo hệ thống co giãn linh hoạt (Elastic Architecture).

### Các công việc triển khai trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu các phân loại cân bằng tải trên AWS: Application Load Balancer (Layer 7 - HTTP/HTTPS), Network Load Balancer (Layer 4 - TCP/UDP), Gateway Load Balancer.<br>- Phân tích các thành phần của ALB: Listeners, Rules, Target Groups, Health Checks.<br>- Khởi tạo hai EC2 Instances tại 2 AZ khác nhau, cấu hình script web hiển thị Hostname và IP để nhận biết server phục vụ. | 07/09/2026 | 07/09/2026 | https://docs.aws.amazon.com/elasticloadbalancing/latest/application/ |
| 3 | - Tạo Target Group giao thức HTTP port 80, cấu hình Health Check đường dẫn `/index.html` với ngưỡng Healthy/Unhealthy threshold.<br>- Đăng ký (Register) 2 EC2 instances vào Target Group.<br>- Khởi tạo Application Load Balancer internet-facing đặt trên 2 Public Subnets của Custom VPC.<br>- Kiểm tra cơ chế cân bằng tải luân phiên (Round-robin) khi reload địa chỉ DNS của ALB. | 08/09/2026 | 08/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| 4 | - Tìm hiểu nguyên lý hoạt động của AWS Auto Scaling Group (ASG): Min Size, Max Size, Desired Capacity.<br>- Tạo Launch Template: định nghĩa AMI, Instance Type `t3.micro`, gán IAM Role, Security Group và mã tự động khởi động User Data script cài đặt web server.<br>- Phân tích sự khác biệt giữa Launch Template (phiên bản mới, linh hoạt) và Launch Configuration (cũ, chuẩn bị ngừng hỗ trợ). | 09/09/2026 | 09/09/2026 | https://docs.aws.amazon.com/autoscaling/ec2/userguide/ |
| 5 | - Khởi tạo Auto Scaling Group liên kết với Launch Template vừa tạo.<br>- Cấu hình ASG trải dài trên 2 Private Subnets (Multi-AZ) và gắn kết trực tiếp với Target Group của ALB.<br>- Thiết lập chính sách co giãn động Target Tracking Scaling Policy: duy trì giá trị `ASGAverageCPUUtilization` ở mức 50%.<br>- Cấu hình thời gian chờ làm nguội (Cooldown Period). | 10/09/2026 | 10/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| 6 | - Kiểm thử tính tự phục hồi (Self-healing): Giả lập sự cố bằng cách cưỡng bức Terminate một EC2 instance, quan sát ASG tự phát hiện thiếu hụt dung lượng và tự động spawn máy chủ mới thay thế.<br>- Tiến hành bài test tải cao bằng lệnh `stress`: Quan sát CloudWatch Alarm kích hoạt ASG tăng số lượng instance (Scale-out) từ 2 lên 4 instances.<br>- Ngừng tải và theo dõi quá trình tự động giảm số lượng máy chủ (Scale-in) an toàn.<br>- Họp đánh giá kỹ thuật và cập nhật tài liệu Hugo. | 11/09/2026 | 11/09/2026 | https://cloudjourney.awsstudygroup.com/ |

### Kết quả đạt được tuần 6:
* **Mức độ hoàn thành:** 100%.
* **Kiến thức lý thuyết:**
  * Nắm vững kiến trúc 3 tầng phân tán (Presentation Tier nằm tại ALB/Public Subnet, Application Tier nằm tại ASG/Private Subnet).
  * Hiểu cơ chế định tuyến thông minh theo Path-based và Host-based Routing của ALB.
  * Hiểu rõ cơ chế tự chữa lành (Self-healing) và tính toán chi phí linh hoạt theo nhu cầu thực tế của ASG.
* **Kỹ năng thực hành:**
  * Triển khai hoàn chỉnh hệ thống Web đa vùng khả dụng (Multi-AZ) đạt chuẩn kiến trúc AWS Well-Architected Framework.
  * Cấu hình thành công kiểm tra sức khỏe tự động để cách ly máy chủ lỗi ra khỏi luồng người dùng.
  * Thiết lập và kiểm thử thực tế thành công cơ chế Scale-out và Scale-in tự động theo ngưỡng tải CPU.