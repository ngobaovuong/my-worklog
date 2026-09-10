---
title: "Đề xuất"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Chợ Đồ Cũ

## Chợ Đồ Cũ Cloud-Native trên AWS

---

# 1. Tóm tắt

Chợ Đồ Cũ là một ứng dụng web trên nền tảng đám mây cho phép người dùng mua và bán sản phẩm đã qua sử dụng thông qua một sàn giao dịch trực tuyến tập trung. Nền tảng cung cấp xác thực người dùng, quản lý sản phẩm, quản lý danh mục, tải lên hình ảnh, tìm kiếm sản phẩm, giỏ hàng, thanh toán và quản lý đơn hàng, đồng thời sử dụng các dịch vụ AWS được quản lý để đảm bảo khả năng mở rộng, tính sẵn sàng, bảo mật và triển khai đơn giản.

Ứng dụng được phát triển bằng **Node.js**, **Express.js**, **MongoDB Atlas** và **EJS**. Ứng dụng được đóng gói bằng **Docker** và triển khai trên **Amazon ECS Fargate** phía sau **Application Load Balancer (ALB)**. Docker Image được lưu trữ trong **Amazon ECR**, hình ảnh sản phẩm được lưu trữ trong **Amazon S3**, và **AWS CodeBuild** tự động build và triển khai phiên bản mới nhất mỗi khi mã nguồn được đẩy lên GitHub.

Môi trường triển khai cũng sử dụng **Amazon Route 53** để quản lý tên miền, **AWS Certificate Manager (ACM)** để mã hóa HTTPS, **Amazon CloudWatch** để giám sát, **AWS IAM** để kiểm soát truy cập và **Amazon VPC** để bảo mật mạng. Kiến trúc này cung cấp triển khai tự động, lưu trữ tập trung, quản lý đơn giản và hạ tầng đám mây có khả năng mở rộng, phù hợp với các ứng dụng thương mại điện tử quy mô nhỏ và vừa.

---

# 2. Vấn đề

## Vấn đề hiện tại

Nhiều chợ đồ cũ hiện nay dựa vào mạng xã hội hoặc các website được quản lý thủ công, khiến việc quản lý sản phẩm kém hiệu quả và khó bảo trì. Hình ảnh sản phẩm thường được lưu trữ cục bộ, việc triển khai yêu cầu cập nhật thủ công và việc mở rộng ứng dụng trở nên khó khăn khi số lượng người dùng tăng.

Các phương pháp triển khai truyền thống cũng làm tăng thời gian ngừng hoạt động, yêu cầu nhiều công sức vận hành hơn và khiến việc bảo trì ứng dụng phức tạp hơn mỗi khi phát hành tính năng mới hoặc sửa lỗi.

## Giải pháp

Giải pháp đề xuất là phát triển một nền tảng chợ đồ cũ cloud-native sử dụng các dịch vụ AWS được quản lý.

Người dùng có thể đăng ký tài khoản, đăng nhập an toàn, tải lên sản phẩm kèm hình ảnh, duyệt sản phẩm theo danh mục, tìm kiếm sản phẩm, quản lý giỏ hàng, đặt hàng và quản lý danh sách sản phẩm của mình thông qua ứng dụng web.

Dữ liệu ứng dụng được lưu trữ trong **MongoDB Atlas**, trong khi hình ảnh sản phẩm được lưu trữ trong **Amazon S3**.

Ứng dụng được đóng gói bằng Docker và triển khai trên **Amazon ECS Fargate**. Mỗi khi mã nguồn được đẩy lên GitHub, **AWS CodeBuild** tự động build Docker Image, đẩy lên **Amazon ECR** và triển khai phiên bản mới nhất lên Amazon ECS.

Giao tiếp HTTPS được bảo mật bằng **AWS Certificate Manager (ACM)** và ứng dụng có thể truy cập thông qua tên miền tùy chỉnh được cấu hình bằng **Amazon Route 53**.

## Lợi ích

Kiến trúc đề xuất mang lại các lợi ích sau:

- Đơn giản hóa triển khai ứng dụng.
- Quy trình CI/CD tự động.
- Hạ tầng đám mây có khả năng mở rộng.
- Giao tiếp HTTPS an toàn.
- Lưu trữ đám mây tin cậy.
- Đơn giản hóa bảo trì ứng dụng.
- Giảm công sức vận hành.
- Dễ dàng mở rộng trong tương lai.
---

# 3. Kiến trúc giải pháp

Ứng dụng sử dụng kiến trúc container cloud-native được triển khai trên các dịch vụ AWS được quản lý.

## Kiến trúc giải pháp

![Kiến trúc hệ thống](/images/2-Proposal/system_architecture.png)

## Các dịch vụ AWS sử dụng

- Amazon VPC
- AWS IAM
- Amazon ECS Fargate
- Amazon ECR
- Amazon S3
- Application Load Balancer (ALB)
- Amazon Route 53
- AWS Certificate Manager (ACM)
- AWS CodeBuild
- Amazon CloudWatch
- MongoDB Atlas

## Thiết kế thành phần

### Frontend

- HTML
- CSS
- JavaScript
- EJS Template Engine

### Backend

- Node.js
- Express.js
- Express Session
- Multer
- AWS SDK for JavaScript

### Cơ sở dữ liệu

- MongoDB Atlas

### Lưu trữ hình ảnh

- Amazon S3

### Nền tảng Container

- Docker
- Amazon ECS Fargate

### Quy trình triển khai

GitHub

↓

AWS CodeBuild

↓

Amazon ECR

↓

Amazon ECS Fargate

---

# 4. Triển khai kỹ thuật

## Các giai đoạn triển khai

Dự án được triển khai qua các giai đoạn sau:

- Nghiên cứu kiến trúc đám mây AWS và chiến lược triển khai.
- Thiết kế kiến trúc tổng thể hệ thống.
- Phát triển Backend bằng Node.js và Express.js.
- Cấu hình MongoDB Atlas cho cơ sở dữ liệu đám mây.
- Tích hợp Amazon S3 để lưu trữ hình ảnh.
- Đóng gói ứng dụng bằng Docker.
- Đẩy Docker Image lên Amazon ECR.
- Triển khai Docker Container trên Amazon ECS Fargate.
- Cấu hình Application Load Balancer.
- Cấu hình Amazon Route 53 và AWS Certificate Manager (ACM).
- Cấu hình AWS CodeBuild để tự động build và triển khai.
- Giám sát ứng dụng bằng Amazon CloudWatch.
- Kiểm thử hệ thống và triển khai môi trường Production.

## Yêu cầu kỹ thuật

### Ngôn ngữ lập trình

- JavaScript
- HTML
- CSS

### Framework

- Express.js
- EJS

### Cơ sở dữ liệu

- MongoDB Atlas

### Dịch vụ đám mây

- Amazon VPC
- AWS IAM
- Amazon ECS Fargate
- Amazon ECR
- Amazon S3
- Application Load Balancer (ALB)
- Amazon Route 53
- AWS Certificate Manager (ACM)
- AWS CodeBuild
- Amazon CloudWatch

### Công cụ phát triển

- Visual Studio Code
- Git
- GitHub
- Docker Desktop
- MongoDB Compass
---

# 5. Lộ trình & Các mốc

Dự án được hoàn thành qua các giai đoạn sau.

### Giai đoạn 1 – Lập kế hoạch dự án

- Phân tích yêu cầu hệ thống.
- Thiết kế kiến trúc tổng thể hệ thống.
- Thiết kế cấu trúc cơ sở dữ liệu MongoDB.
- Chuẩn bị môi trường phát triển.

### Giai đoạn 2 – Phát triển ứng dụng

- Phát triển xác thực người dùng.
- Phát triển chức năng khách hàng.
- Phát triển chức năng cửa hàng.
- Phát triển chức năng quản trị viên.
- Phát triển quản lý sản phẩm.
- Phát triển quản lý đơn hàng.

### Giai đoạn 3 – Tích hợp đám mây

- Cấu hình MongoDB Atlas.
- Tích hợp Amazon S3 để lưu trữ hình ảnh.
- Kiểm tra kết nối lưu trữ đám mây.

### Giai đoạn 4 – Đóng gói Container

- Tạo Dockerfile.
- Build Docker Image.
- Kiểm tra Docker Container cục bộ.

### Giai đoạn 5 – Triển khai AWS

- Đẩy Docker Image lên Amazon ECR.
- Triển khai ứng dụng lên Amazon ECS Fargate.
- Cấu hình Application Load Balancer.
- Cấu hình Amazon Route 53.
- Cấu hình AWS Certificate Manager (ACM).

### Giai đoạn 6 – CI/CD

- Kết nối kho GitHub.
- Cấu hình AWS CodeBuild.
- Tự động triển khai ứng dụng.

### Giai đoạn 7 – Giám sát & Kiểm thử

- Cấu hình Amazon CloudWatch.
- Thực hiện kiểm thử chức năng.
- Xác minh triển khai ứng dụng.
- Khắc phục lỗi triển khai.

### Giai đoạn 8 – Hoàn thành dự án

- Triển khai môi trường Production.
- Hoàn thiện tài liệu.
- Trình bày dự án hoàn thành.

---

# 6. Ước tính chi phí

## Ước tính chi phí hạ tầng

| Dịch vụ | Chi phí ước tính |
|----------|------------------|
| Amazon ECS Fargate | ~0.25 USD/tháng |
| Amazon S3 (Lưu trữ & Requests) | ~0.15 USD/tháng |
| Amazon ECR | ~0.03 USD/tháng |
| AWS CodeBuild | ~0.05 USD/tháng |
| Application Load Balancer | ~0.10 USD/tháng |
| Amazon CloudWatch | ~0.02 USD/tháng |
| **Tổng ước tính** | **~0.60 USD/tháng** |

### Hướng dẫn kiểm soát chi phí

- **AWS Budgets:** Cảnh báo tự động khi chi phí vượt **5.00 USD** và **10.00 USD**.
- **Amazon ECR Lifecycle Policy:** Tự động xóa Docker Image không sử dụng.
- **AWS CodeBuild:** Chỉ build khi mã nguồn được đẩy lên kho GitHub.
- **Dọn dẹp sau demo:** Xóa ECS services, ECR images, S3 objects không sử dụng, Application Load Balancer, CloudWatch alarms, ACM certificates và Route 53 hosted zones sau khi hoàn thành dự án để tránh chi phí không cần thiết.
---

# 7. Đánh giá rủi ro

## Ma trận rủi ro

- Triển khai Amazon ECS thất bại.
- Lỗi kết nối MongoDB Atlas.
- Tải lên Amazon S3 thất bại.
- AWS CodeBuild build thất bại.
- Lỗi cấu hình DNS Route 53.
- Lỗi cấu hình chứng chỉ HTTPS.
- Chi phí dịch vụ AWS ngoài dự kiến.

## Chiến lược giảm thiểu

- Bật giám sát Amazon CloudWatch.
- Cấu hình cảnh báo AWS Budgets.
- Quản lý phiên bản Docker Image bằng Amazon ECR.
- Sao lưu MongoDB Atlas định kỳ.
- Áp dụng chính sách IAM theo nguyên tắc đặc quyền tối thiểu.
- Kiểm tra bản ghi DNS Route 53 trước khi triển khai.
- Kiểm tra trạng thái chứng chỉ ACM trước khi bật HTTPS.

## Kế hoạch dự phòng

- Khôi phục Docker Image trước đó.
- Triển khai lại Amazon ECS Task Definition trước đó.
- Khôi phục bản sao lưu MongoDB Atlas.
- Triển khai lại qua AWS CodeBuild.
- Cấu hình lại bản ghi DNS Route 53 nếu cần.
- Cấp lại chứng chỉ ACM khi xác thực thất bại.

---

# 8. Kết quả mong đợi

## Kết quả kỹ thuật

Dự án hoàn thành sẽ cung cấp:

- Nền tảng chợ đồ cũ cloud-native được container hóa hoàn chỉnh.
- Triển khai CI/CD tự động bằng GitHub và AWS CodeBuild.
- Lưu trữ hình ảnh tin cậy bằng Amazon S3.
- Triển khai container có khả năng mở rộng bằng Amazon ECS Fargate.
- Giao tiếp HTTPS an toàn bằng AWS Certificate Manager (ACM).
- Quản lý tên miền tùy chỉnh bằng Amazon Route 53.
- Cân bằng tải bằng Application Load Balancer.
- Cơ sở dữ liệu đám mây tập trung bằng MongoDB Atlas.
- Giám sát tài nguyên bằng Amazon CloudWatch.
- Quản lý truy cập an toàn bằng AWS IAM.

## Giá trị kinh doanh

Dự án minh họa việc triển khai thực tế điện toán đám mây, container hóa và DevOps bằng các dịch vụ AWS được quản lý.

Kiến trúc cloud-native giúp đơn giản hóa triển khai, giảm công sức vận hành, cải thiện khả năng mở rộng và cung cấp nền tảng tin cậy cho việc mở rộng trong tương lai.

Các cải tiến trong tương lai có thể bao gồm tích hợp thanh toán trực tuyến, hệ thống gợi ý, dịch vụ thông báo, bảng điều khiển phân tích và kiến trúc microservice, đồng thời vẫn duy trì tính sẵn sàng cao và hiệu quả vận hành.