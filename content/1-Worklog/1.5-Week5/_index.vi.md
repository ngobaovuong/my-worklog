---
title: "Worklog Tuần 5"
date: 2026-08-31
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:
* Nghiên cứu toàn diện về văn hóa giám sát vận hành (Observability) trên hạ tầng AWS.
* Sử dụng Amazon CloudWatch để thu thập chỉ số hiệu năng (Metrics), lưu trữ log tập trung (Logs) và thiết lập cảnh báo chủ động (Alarms).
* Cấu hình Amazon Simple Notification Service (Amazon SNS) để phân phối thông báo sự cố tức thời qua Email.
* Khai thác AWS CloudTrail để ghi nhận và truy vết toàn bộ các lệnh gọi API phục vụ kiểm toán an ninh và truy cứu nguyên nhân gốc rễ sự cố (RCA).

### Các công việc triển khai trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu 3 trụ cột của Observability: Metrics, Logs và Traces.<br>- Phân tích các chỉ số mặc định của Amazon EC2 (CPUUtilization, DiskReadBytes, NetworkIn, StatusCheckFailed).<br>- Tìm hiểu sự khác biệt giữa Basic Monitoring (chu kỳ 5 phút, miễn phí) và Detailed Monitoring (chu kỳ 1 phút, có tính phí). | 31/08/2026 | 31/08/2026 | https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ |
| 3 | - Xây dựng CloudWatch Custom Dashboard: tạo các widget dạng đường kẻ (Line Chart), dạng đồng hồ (Number/Gauge) để giám sát tài nguyên máy chủ.<br>- Cài đặt và cấu hình CloudWatch Unified Agent trên máy chủ EC2 Linux.<br>- Thu thập bổ sung các chỉ số cấp hệ điều hành chuyên sâu mà AWS hypervisor không thấy được: Memory Usage (RAM) và Disk Space Utilization. | 01/09/2026 | 01/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| 4 | - Tìm hiểu dịch vụ gửi thông báo đẩy Amazon SNS (Simple Notification Service).<br>- Tạo một SNS Standard Topic tên `DevOps-Alerts`, thêm Subscription loại Email và xác thực subscription qua email cá nhân.<br>- Tạo CloudWatch Metric Alarm: Cảnh báo trạng thái cảnh báo khi `CPUUtilization >= 75%` liên tục trong 2 chu kỳ đánh giá (Evaluation Periods = 2/2).<br>- Liên kết trạng thái ALARM với SNS Topic vừa tạo. | 02/09/2026 | 02/09/2026 | https://docs.aws.amazon.com/sns/latest/dg/ |
| 5 | - Cài đặt gói công cụ `stress` trên máy ảo EC2 để ép tải CPU lên 100%: `stress --cpu 2 --timeout 300s`.<br>- Quan sát sự biến đổi đồ thị trên CloudWatch Dashboard, chứng kiến Alarm chuyển từ trạng thái `OK` sang `ALARM`.<br>- Kiểm tra hộp thư nhận email thông báo cảnh báo tự động gửi về từ SNS.<br>- Quan sát hệ thống tự phục hồi về trạng thái `OK` khi stress test kết thúc. | 03/09/2026 | 03/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| 6 | - Tìm hiểu AWS CloudTrail: Phân biệt Management Events, Data Events và Insights Events.<br>- Khởi tạo Multi-Region Trail lưu log vào một S3 Bucket bảo mật được bật mã hóa SSE-KMS.<br>- Sử dụng công cụ CloudTrail Event History để điều tra sự cố giả định: xác định danh tính (IAM User, IP gọi API, User Agent) của người vừa thay đổi Security Group hoặc khởi chạy EC2.<br>- Tổng kết kiến thức tuần 5. | 04/09/2026 | 04/09/2026 | https://docs.aws.amazon.com/awscloudtrail/latest/userguide/ |

### Kết quả đạt được tuần 5:
* **Mức độ hoàn thành:** 100%.
* **Kiến thức lý thuyết:**
  * Hiểu cơ chế phân tách giữa giám sát phần cứng vật lý và giám sát bên trong OS (Memory, Disk).
  * Hiểu nguyên lý phân phối Pub/Sub (Publisher - Subscriber) của dịch vụ Amazon SNS.
  * Nắm vững cách CloudTrail hỗ trợ tuân thủ chuẩn kiểm toán (Compliance & Auditing) và điều tra phản ứng sự cố bảo mật (Incident Response).
* **Kỹ năng thực hành:**
  * Triển khai thành công CloudWatch Agent theo dõi mức chiếm dụng RAM và dung lượng ổ cứng.
  * Xây dựng hệ thống tự động phát hiện quá tải và phát tín hiệu cảnh báo qua Email.
  * Thành thạo kỹ năng phân tích JSON log sự kiện trong CloudTrail để truy vết ai đã làm gì, vào thời điểm nào trên hạ tầng AWS.---
title: "Worklog Tuần 5"
date: 2026-08-31
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:
* Nghiên cứu toàn diện về văn hóa giám sát vận hành (Observability) trên hạ tầng AWS.
* Sử dụng Amazon CloudWatch để thu thập chỉ số hiệu năng (Metrics), lưu trữ log tập trung (Logs) và thiết lập cảnh báo chủ động (Alarms).
* Cấu hình Amazon Simple Notification Service (Amazon SNS) để phân phối thông báo sự cố tức thời qua Email.
* Khai thác AWS CloudTrail để ghi nhận và truy vết toàn bộ các lệnh gọi API phục vụ kiểm toán an ninh và truy cứu nguyên nhân gốc rễ sự cố (RCA).

### Các công việc triển khai trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu 3 trụ cột của Observability: Metrics, Logs và Traces.<br>- Phân tích các chỉ số mặc định của Amazon EC2 (CPUUtilization, DiskReadBytes, NetworkIn, StatusCheckFailed).<br>- Tìm hiểu sự khác biệt giữa Basic Monitoring (chu kỳ 5 phút, miễn phí) và Detailed Monitoring (chu kỳ 1 phút, có tính phí). | 31/08/2026 | 31/08/2026 | https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ |
| 3 | - Xây dựng CloudWatch Custom Dashboard: tạo các widget dạng đường kẻ (Line Chart), dạng đồng hồ (Number/Gauge) để giám sát tài nguyên máy chủ.<br>- Cài đặt và cấu hình CloudWatch Unified Agent trên máy chủ EC2 Linux.<br>- Thu thập bổ sung các chỉ số cấp hệ điều hành chuyên sâu mà AWS hypervisor không thấy được: Memory Usage (RAM) và Disk Space Utilization. | 01/09/2026 | 01/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| 4 | - Tìm hiểu dịch vụ gửi thông báo đẩy Amazon SNS (Simple Notification Service).<br>- Tạo một SNS Standard Topic tên `DevOps-Alerts`, thêm Subscription loại Email và xác thực subscription qua email cá nhân.<br>- Tạo CloudWatch Metric Alarm: Cảnh báo trạng thái cảnh báo khi `CPUUtilization >= 75%` liên tục trong 2 chu kỳ đánh giá (Evaluation Periods = 2/2).<br>- Liên kết trạng thái ALARM với SNS Topic vừa tạo. | 02/09/2026 | 02/09/2026 | https://docs.aws.amazon.com/sns/latest/dg/ |
| 5 | - Cài đặt gói công cụ `stress` trên máy ảo EC2 để ép tải CPU lên 100%: `stress --cpu 2 --timeout 300s`.<br>- Quan sát sự biến đổi đồ thị trên CloudWatch Dashboard, chứng kiến Alarm chuyển từ trạng thái `OK` sang `ALARM`.<br>- Kiểm tra hộp thư nhận email thông báo cảnh báo tự động gửi về từ SNS.<br>- Quan sát hệ thống tự phục hồi về trạng thái `OK` khi stress test kết thúc. | 03/09/2026 | 03/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| 6 | - Tìm hiểu AWS CloudTrail: Phân biệt Management Events, Data Events và Insights Events.<br>- Khởi tạo Multi-Region Trail lưu log vào một S3 Bucket bảo mật được bật mã hóa SSE-KMS.<br>- Sử dụng công cụ CloudTrail Event History để điều tra sự cố giả định: xác định danh tính (IAM User, IP gọi API, User Agent) của người vừa thay đổi Security Group hoặc khởi chạy EC2.<br>- Tổng kết kiến thức tuần 5. | 04/09/2026 | 04/09/2026 | https://docs.aws.amazon.com/awscloudtrail/latest/userguide/ |

### Kết quả đạt được tuần 5:
* **Mức độ hoàn thành:** 100%.
* **Kiến thức lý thuyết:**
  * Hiểu cơ chế phân tách giữa giám sát phần cứng vật lý và giám sát bên trong OS (Memory, Disk).
  * Hiểu nguyên lý phân phối Pub/Sub (Publisher - Subscriber) của dịch vụ Amazon SNS.
  * Nắm vững cách CloudTrail hỗ trợ tuân thủ chuẩn kiểm toán (Compliance & Auditing) và điều tra phản ứng sự cố bảo mật (Incident Response).
* **Kỹ năng thực hành:**
  * Triển khai thành công CloudWatch Agent theo dõi mức chiếm dụng RAM và dung lượng ổ cứng.
  * Xây dựng hệ thống tự động phát hiện quá tải và phát tín hiệu cảnh báo qua Email.
  * Thành thạo kỹ năng phân tích JSON log sự kiện trong CloudTrail để truy vết ai đã làm gì, vào thời điểm nào trên hạ tầng AWS.---
title: "Worklog Tuần 5"
date: 2026-08-31
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:
* Nghiên cứu toàn diện về văn hóa giám sát vận hành (Observability) trên hạ tầng AWS.
* Sử dụng Amazon CloudWatch để thu thập chỉ số hiệu năng (Metrics), lưu trữ log tập trung (Logs) và thiết lập cảnh báo chủ động (Alarms).
* Cấu hình Amazon Simple Notification Service (Amazon SNS) để phân phối thông báo sự cố tức thời qua Email.
* Khai thác AWS CloudTrail để ghi nhận và truy vết toàn bộ các lệnh gọi API phục vụ kiểm toán an ninh và truy cứu nguyên nhân gốc rễ sự cố (RCA).

### Các công việc triển khai trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu 3 trụ cột của Observability: Metrics, Logs và Traces.<br>- Phân tích các chỉ số mặc định của Amazon EC2 (CPUUtilization, DiskReadBytes, NetworkIn, StatusCheckFailed).<br>- Tìm hiểu sự khác biệt giữa Basic Monitoring (chu kỳ 5 phút, miễn phí) và Detailed Monitoring (chu kỳ 1 phút, có tính phí). | 31/08/2026 | 31/08/2026 | https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ |
| 3 | - Xây dựng CloudWatch Custom Dashboard: tạo các widget dạng đường kẻ (Line Chart), dạng đồng hồ (Number/Gauge) để giám sát tài nguyên máy chủ.<br>- Cài đặt và cấu hình CloudWatch Unified Agent trên máy chủ EC2 Linux.<br>- Thu thập bổ sung các chỉ số cấp hệ điều hành chuyên sâu mà AWS hypervisor không thấy được: Memory Usage (RAM) và Disk Space Utilization. | 01/09/2026 | 01/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| 4 | - Tìm hiểu dịch vụ gửi thông báo đẩy Amazon SNS (Simple Notification Service).<br>- Tạo một SNS Standard Topic tên `DevOps-Alerts`, thêm Subscription loại Email và xác thực subscription qua email cá nhân.<br>- Tạo CloudWatch Metric Alarm: Cảnh báo trạng thái cảnh báo khi `CPUUtilization >= 75%` liên tục trong 2 chu kỳ đánh giá (Evaluation Periods = 2/2).<br>- Liên kết trạng thái ALARM với SNS Topic vừa tạo. | 02/09/2026 | 02/09/2026 | https://docs.aws.amazon.com/sns/latest/dg/ |
| 5 | - Cài đặt gói công cụ `stress` trên máy ảo EC2 để ép tải CPU lên 100%: `stress --cpu 2 --timeout 300s`.<br>- Quan sát sự biến đổi đồ thị trên CloudWatch Dashboard, chứng kiến Alarm chuyển từ trạng thái `OK` sang `ALARM`.<br>- Kiểm tra hộp thư nhận email thông báo cảnh báo tự động gửi về từ SNS.<br>- Quan sát hệ thống tự phục hồi về trạng thái `OK` khi stress test kết thúc. | 03/09/2026 | 03/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| 6 | - Tìm hiểu AWS CloudTrail: Phân biệt Management Events, Data Events và Insights Events.<br>- Khởi tạo Multi-Region Trail lưu log vào một S3 Bucket bảo mật được bật mã hóa SSE-KMS.<br>- Sử dụng công cụ CloudTrail Event History để điều tra sự cố giả định: xác định danh tính (IAM User, IP gọi API, User Agent) của người vừa thay đổi Security Group hoặc khởi chạy EC2.<br>- Tổng kết kiến thức tuần 5. | 04/09/2026 | 04/09/2026 | https://docs.aws.amazon.com/awscloudtrail/latest/userguide/ |

### Kết quả đạt được tuần 5:
* **Mức độ hoàn thành:** 100%.
* **Kiến thức lý thuyết:**
  * Hiểu cơ chế phân tách giữa giám sát phần cứng vật lý và giám sát bên trong OS (Memory, Disk).
  * Hiểu nguyên lý phân phối Pub/Sub (Publisher - Subscriber) của dịch vụ Amazon SNS.
  * Nắm vững cách CloudTrail hỗ trợ tuân thủ chuẩn kiểm toán (Compliance & Auditing) và điều tra phản ứng sự cố bảo mật (Incident Response).
* **Kỹ năng thực hành:**
  * Triển khai thành công CloudWatch Agent theo dõi mức chiếm dụng RAM và dung lượng ổ cứng.
  * Xây dựng hệ thống tự động phát hiện quá tải và phát tín hiệu cảnh báo qua Email.
  * Thành thạo kỹ năng phân tích JSON log sự kiện trong CloudTrail để truy vết ai đã làm gì, vào thời điểm nào trên hạ tầng AWS.---
title: "Worklog Tuần 5"
date: 2026-08-31
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:
* Nghiên cứu toàn diện về văn hóa giám sát vận hành (Observability) trên hạ tầng AWS.
* Sử dụng Amazon CloudWatch để thu thập chỉ số hiệu năng (Metrics), lưu trữ log tập trung (Logs) và thiết lập cảnh báo chủ động (Alarms).
* Cấu hình Amazon Simple Notification Service (Amazon SNS) để phân phối thông báo sự cố tức thời qua Email.
* Khai thác AWS CloudTrail để ghi nhận và truy vết toàn bộ các lệnh gọi API phục vụ kiểm toán an ninh và truy cứu nguyên nhân gốc rễ sự cố (RCA).

### Các công việc triển khai trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu 3 trụ cột của Observability: Metrics, Logs và Traces.<br>- Phân tích các chỉ số mặc định của Amazon EC2 (CPUUtilization, DiskReadBytes, NetworkIn, StatusCheckFailed).<br>- Tìm hiểu sự khác biệt giữa Basic Monitoring (chu kỳ 5 phút, miễn phí) và Detailed Monitoring (chu kỳ 1 phút, có tính phí). | 31/08/2026 | 31/08/2026 | https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ |
| 3 | - Xây dựng CloudWatch Custom Dashboard: tạo các widget dạng đường kẻ (Line Chart), dạng đồng hồ (Number/Gauge) để giám sát tài nguyên máy chủ.<br>- Cài đặt và cấu hình CloudWatch Unified Agent trên máy chủ EC2 Linux.<br>- Thu thập bổ sung các chỉ số cấp hệ điều hành chuyên sâu mà AWS hypervisor không thấy được: Memory Usage (RAM) và Disk Space Utilization. | 01/09/2026 | 01/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| 4 | - Tìm hiểu dịch vụ gửi thông báo đẩy Amazon SNS (Simple Notification Service).<br>- Tạo một SNS Standard Topic tên `DevOps-Alerts`, thêm Subscription loại Email và xác thực subscription qua email cá nhân.<br>- Tạo CloudWatch Metric Alarm: Cảnh báo trạng thái cảnh báo khi `CPUUtilization >= 75%` liên tục trong 2 chu kỳ đánh giá (Evaluation Periods = 2/2).<br>- Liên kết trạng thái ALARM với SNS Topic vừa tạo. | 02/09/2026 | 02/09/2026 | https://docs.aws.amazon.com/sns/latest/dg/ |
| 5 | - Cài đặt gói công cụ `stress` trên máy ảo EC2 để ép tải CPU lên 100%: `stress --cpu 2 --timeout 300s`.<br>- Quan sát sự biến đổi đồ thị trên CloudWatch Dashboard, chứng kiến Alarm chuyển từ trạng thái `OK` sang `ALARM`.<br>- Kiểm tra hộp thư nhận email thông báo cảnh báo tự động gửi về từ SNS.<br>- Quan sát hệ thống tự phục hồi về trạng thái `OK` khi stress test kết thúc. | 03/09/2026 | 03/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| 6 | - Tìm hiểu AWS CloudTrail: Phân biệt Management Events, Data Events và Insights Events.<br>- Khởi tạo Multi-Region Trail lưu log vào một S3 Bucket bảo mật được bật mã hóa SSE-KMS.<br>- Sử dụng công cụ CloudTrail Event History để điều tra sự cố giả định: xác định danh tính (IAM User, IP gọi API, User Agent) của người vừa thay đổi Security Group hoặc khởi chạy EC2.<br>- Tổng kết kiến thức tuần 5. | 04/09/2026 | 04/09/2026 | https://docs.aws.amazon.com/awscloudtrail/latest/userguide/ |

### Kết quả đạt được tuần 5:
* **Mức độ hoàn thành:** 100%.
* **Kiến thức lý thuyết:**
  * Hiểu cơ chế phân tách giữa giám sát phần cứng vật lý và giám sát bên trong OS (Memory, Disk).
  * Hiểu nguyên lý phân phối Pub/Sub (Publisher - Subscriber) của dịch vụ Amazon SNS.
  * Nắm vững cách CloudTrail hỗ trợ tuân thủ chuẩn kiểm toán (Compliance & Auditing) và điều tra phản ứng sự cố bảo mật (Incident Response).
* **Kỹ năng thực hành:**
  * Triển khai thành công CloudWatch Agent theo dõi mức chiếm dụng RAM và dung lượng ổ cứng.
  * Xây dựng hệ thống tự động phát hiện quá tải và phát tín hiệu cảnh báo qua Email.
  * Thành thạo kỹ năng phân tích JSON log sự kiện trong CloudTrail để truy vết ai đã làm gì, vào thời điểm nào trên hạ tầng AWS.