---
title: "Worklog Tuần 1"
date: 2026-08-03
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu tuần 1:
* Tham gia buổi định hướng thực tập, nắm rõ nội quy, quy trình làm việc và lộ trình đào tạo Cloud Foundation.
* Nắm vững các khái niệm cơ bản về điện toán đám mây (IaaS, PaaS, SaaS) và mô hình hạ tầng toàn cầu của AWS (Regions, Availability Zones, Edge Locations).
* Đăng ký thành công tài khoản AWS Free Tier và thiết lập các lớp phòng thủ an ninh tài khoản ban đầu.
* Cài đặt, cấu hình và sử dụng AWS Command Line Interface (AWS CLI v2) trên máy trạm cá nhân.
* Hoàn thành xuất sắc 5 nhiệm vụ khởi động thực hành (đạt điểm số tuyệt đối 200/200).

### Các công việc triển khai trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tham gia buổi định hướng thực tập (Internship Orientation), nhận bàn giao tài liệu hướng dẫn và làm quen với mentor.<br>- Tìm hiểu tổng quan về kiến trúc nền tảng AWS, mô hình chia sẻ trách nhiệm (Shared Responsibility Model).<br>- Phân tích sự khác biệt giữa hạ tầng tại chỗ (On-Premises) và điện toán đám mây công cộng (Public Cloud). | 03/08/2026 | 03/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| 3 | - Tiến hành đăng ký tài khoản AWS cá nhân diện Free Tier.<br>- Kích hoạt bảo mật xác thực đa yếu tố (MFA qua Virtual Authenticator app) cho tài khoản Root.<br>- Xóa bỏ Root Access Keys nhằm triệt tiêu nguy cơ rò rỉ đặc quyền tối cao.<br>- Thiết lập AWS Budgets và CloudWatch Billing Alarm để cảnh báo khi chi phí chạm ngưỡng 1 USD. | 04/08/2026 | 04/08/2026 | https://docs.aws.amazon.com/accounts/latest/reference/ |
| 4 | - Tìm hiểu sâu về AWS Global Infrastructure: phân tích độ trễ giữa Region Singapore (ap-southeast-1), Tokyo (ap-northeast-1) và Sydney (ap-southeast-2).<br>- Thực hiện Nhiệm vụ 1: Khám phá AWS Management Console, chuyển đổi linh hoạt giữa các Region.<br>- Thực hiện Nhiệm vụ 2: Khởi tạo IAM Admin User riêng biệt, phân quyền nhóm và vô hiệu hóa việc dùng tài khoản Root hằng ngày. | 05/08/2026 | 05/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| 5 | - Cài đặt AWS CLI v2 trên hệ điều hành cục bộ (Linux/macOS/Windows).<br>- Tạo Access Key ID và Secret Access Key cho IAM User, cấu hình `aws configure` với định dạng output JSON và default region `ap-southeast-1`.<br>- Thực hiện Nhiệm vụ 3: Sử dụng các câu lệnh kiểm tra danh tính `aws sts get-caller-identity`.<br>- Thực hiện Nhiệm vụ 4: Truy vấn danh sách dịch vụ và vùng khả dụng `aws ec2 describe-regions`. | 06/08/2026 | 06/08/2026 | https://docs.aws.amazon.com/cli/latest/userguide/ |
| 6 | - Thực hiện Nhiệm vụ 5: Kiểm tra chính sách truy cập và xác thực danh quyền hệ thống qua IAM Policy Simulator.<br>- Tổng kết hoàn thành đủ 5 nhiệm vụ khởi động với kết quả tuyệt đối 200/200 điểm.<br>- Viết báo cáo tổng kết tuần 1, cập nhật tài liệu Hugo Markdown và họp nghiệm thu tiến độ với đơn vị hướng dẫn (ĐVHD). | 07/08/2026 | 07/08/2026 | https://cloudjourney.awsstudygroup.com/ |

### Kết quả đạt được tuần 1:
* **Mức độ hoàn thành:** 100%.
* **Kiến thức lý thuyết:**
  * Nắm vững 6 lợi ích cốt lõi của Cloud Computing (đổi chi phí cố định thành biến đổi, lợi thế kinh tế theo quy mô, tốc độ triển khai, loại bỏ phỏng đoán dung lượng, tập trung vào kinh doanh cốt lõi, tiếp cận toàn cầu trong vài phút).
  * Hiểu rõ mô hình phân chia trách nhiệm: AWS chịu trách nhiệm bảo mật "of the Cloud" (phần cứng, hạ tầng mạng, máy chủ vật lý), người dùng chịu trách nhiệm bảo mật "in the Cloud" (dữ liệu, cấu hình IAM, hệ điều hành, firewall).
* **Kỹ năng thực hành:**
  * Thiết lập tài khoản AWS đạt tiêu chuẩn an ninh cơ sở CIS AWS Foundations Benchmark (bật Root MFA, khóa Root API Keys, kích hoạt cảnh báo chi phí).
  * Sử dụng thành thạo AWS CLI v2 để tương tác với tài nguyên đám mây thay vì chỉ thao tác Console giao diện.
  * Hoàn thành 100% nhiệm vụ thử thách ban đầu, đạt điểm tối đa 200/200 điểm.