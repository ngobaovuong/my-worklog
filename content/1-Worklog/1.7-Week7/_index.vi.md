---
title: "Worklog Tuần 7"
date: 2026-09-14
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:
* Nắm vững kiến thức nền tảng về công nghệ Container hóa với Docker (Images, Containers, Layers, Multi-stage build).
* Làm quen với kho lưu trữ container chuyên dụng Amazon Elastic Container Registry (Amazon ECR).
* Nghiên cứu dịch vụ điều phối container Amazon Elastic Container Service (Amazon ECS).
* Phân biệt và ứng dụng hai mô hình tính toán: ECS EC2 Launch Type và AWS Fargate (Serverless Container).
* Xây dựng, đẩy image lên kho lưu trữ và triển khai dịch vụ ứng dụng web hoàn chỉnh trên nền tảng AWS Fargate.

### Các công việc triển khai trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Ôn tập kiến thức Docker: so sánh sự khác biệt về kích thước, tốc độ khởi động và kiến trúc tài nguyên giữa Virtual Machine (Hypervisor) và Container (Shared OS Kernel).<br>- Viết Dockerfile đóng gói một ứng dụng Node.js/Python Web App: sử dụng base image siêu nhẹ `alpine`, tối ưu hóa layer caching.<br>- Build và chạy thử nghiệm container ở máy local: `docker build -t web-app:v1 .` và `docker run -d -p 8080:80 web-app:v1`. | 14/09/2026 | 14/09/2026 | https://docs.docker.com/get-started/ |
| 3 | - Tìm hiểu Amazon Elastic Container Registry (ECR) làm kho chứa image tập trung an toàn.<br>- Khởi tạo Private ECR Repository tên `cloud-demo-app`, bật tính năng Image Scanning on push để tự động quét lỗ hổng bảo mật CVE.<br>- Xác thực Docker CLI với AWS ECR bằng AWS CLI: `aws ecr get-login-password \| docker login ...`.<br>- Gắn tag phiên bản và đẩy (push) image lên kho lưu trữ ECR. | 15/09/2026 | 15/09/2026 | https://docs.aws.amazon.com/AmazonECR/latest/userguide/ |
| 4 | - Tìm hiểu kiến trúc tổng thể của AWS ECS: ECS Cluster, Task Definition, Service, Tasks.<br>- Phân tích chuyên sâu sự khác biệt: ECS với EC2 Launch Type (người dùng tự quản lý máy chủ nền tảng, cài ECS Agent) và AWS Fargate (AWS quản lý toàn bộ hạ tầng máy chủ ngầm, người dùng chỉ định nghĩa CPU/RAM cho từng Task).<br>- Phân biệt 2 loại IAM Role trong ECS: Task Execution Role (kéo image từ ECR, gửi log về CloudWatch) và Task Role (cho phép chính ứng dụng bên trong container gọi các API của AWS như S3, DynamoDB). | 16/09/2026 | 16/09/2026 | https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ |
| 5 | - Soạn thảo một ECS Task Definition: chỉ định compatibility `FARGATE`, cấp phát 0.5 vCPU và 1 GB RAM, chỉ định image URI từ ECR.<br>- Cấu hình log driver `awslogs` để tự động đẩy toàn bộ stdout/stderr của container vào CloudWatch Log Group.<br>- Khởi tạo một ECS Cluster trống dành riêng cho Fargate. | 17/09/2026 | 17/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| 6 | - Khởi tạo một ECS Service: thiết lập số lượng Desired Tasks = 2, chọn mô hình mạng `awsvpc`, đặt trong các Subnet của VPC.<br>- Cấu hình Security Group cho Task: chỉ mở port 80 cho phép truy cập.<br>- Liên kết ECS Service với Application Load Balancer để tiếp nhận và phân phối lưu lượng truy cập trực tiếp vào các Tasks.<br>- Kiểm tra trạng thái hoạt động của Tasks, truy cập thành công ứng dụng qua ALB DNS name.<br>- Viết báo cáo thực hành chi tiết tuần 7. | 18/09/2026 | 18/09/2026 | https://cloudjourney.awsstudygroup.com/ |

### Kết quả đạt được tuần 7:
* **Mức độ hoàn thành:** 100%.
* **Kiến thức lý thuyết:**
  * Nắm vững chu trình phát triển ứng dụng hiện đại theo hướng Containerization.
  * Hiểu rõ ưu thế của kiến trúc Serverless Container (AWS Fargate) trong việc loại bỏ hoàn toàn gánh nặng bảo trì, vá lỗi hệ điều hành và quản lý cụm máy chủ ảo.
  * Hiểu cơ chế mạng `awsvpc` nơi mỗi task container được gán trực tiếp một card mạng đàn hồi riêng (Elastic Network Interface - ENI) cùng địa chỉ IP độc lập.
* **Kỹ năng thực hành:**
  * Đóng gói chuẩn hóa ứng dụng thành Docker Image có tính di động cao.
  * Thiết lập pipeline đẩy image an toàn lên kho lưu trữ Amazon ECR.
  * Triển khai và điều phối thành công dịch vụ web container có khả năng tự duy trì số lượng task trên AWS ECS Fargate tích hợp cùng Application Load Balancer.