---
title: "Nhật ký công việc"
date: 2026-08-03
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

# Nhật ký công việc

Phần này trình bày toàn bộ quá trình thực tập chuyên môn trong **8 tuần** (từ ngày **03/08/2026** đến ngày **25/09/2026**). Trong suốt thời gian thực tập, tôi đã tuân thủ lộ trình đào tạo và thực hành có hệ thống về Điện toán đám mây với nền tảng Amazon Web Services (AWS), đi từ những khái niệm cốt lõi đến việc thiết kế, vận hành, giám sát và tối ưu hóa hạ tầng ứng dụng hiện đại.

Lộ trình thực tập được phân bổ khoa học qua các giai đoạn:
* **Nền tảng và Dịch vụ cốt lõi (Tuần 1 - 3):** Tìm hiểu tổng quan kiến trúc AWS, thiết lập an toàn tài khoản với IAM, triển khai máy ảo EC2, lưu trữ đối tượng S3 và xây dựng kiến trúc mạng cô lập với Amazon VPC (chia Subnet Public/Private, Internet Gateway, định tuyến Route Tables).
* **Điện toán hiện đại và Vận hành giám sát (Tuần 4 - 5):** Làm quen với kiến trúc Serverless qua AWS Lambda kích hoạt theo sự kiện (Event-driven); thiết lập hệ thống quan sát vận hành (Observability) toàn diện với Amazon CloudWatch (Metrics, Dashboards, Alarms), gửi cảnh báo qua Amazon SNS và kiểm toán an ninh với AWS CloudTrail.
* **Khả năng co giãn và Đóng gói Container (Tuần 6 - 7):** Xây dựng hạ tầng có tính sẵn sàng cao (High Availability) bằng cách kết hợp Elastic Load Balancer (ALB) và Auto Scaling Groups (ASG); tìm hiểu Docker, tối ưu hóa image, lưu trữ trên Amazon ECR và điều phối dịch vụ container trên Amazon ECS Fargate.
* **Tổng kết và Nghiệm thu (Tuần 8):** Hệ thống hóa kiến trúc giải pháp theo chuẩn AWS Well-Architected Framework, thực hiện quy trình dọn dẹp tài nguyên an toàn nhằm tối ưu chi phí và báo cáo kết quả thực tập với Đơn vị Hướng dẫn (ĐVHD).

Nội dung chi tiết của từng tuần được trình bày cụ thể như sau:

* **Tuần 1:** [Tổng quan kiến trúc AWS và lập tài khoản thực hành](1.1-week1/)
* **Tuần 2:** [AWS Core Services: EC2, S3 và IAM](1.2-week2/)
* **Tuần 3:** [AWS Networking: Amazon VPC, Subnet và Internet Gateway](1.3-week3/)
* **Tuần 4:** [AWS Lambda và mô hình Serverless](1.4-week4/)
* **Tuần 5:** [Giám sát hệ thống với CloudWatch và kiểm vết với CloudTrail](1.5-week5/)
* **Tuần 6:** [Cân bằng tải Elastic Load Balancer (ELB) và Auto Scaling](1.6-week6/)
* **Tuần 7:** [Công nghệ Docker và điều phối container với Amazon ECS Fargate](1.7-week7/)
* **Tuần 8:** [Tổng kết kỳ thực tập, dọn dẹp tài nguyên và đánh giá kết quả](1.8-week8/)