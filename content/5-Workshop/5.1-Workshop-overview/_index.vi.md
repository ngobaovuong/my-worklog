---
title : "Tổng quan Workshop"
date : 2026-09-25
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

### Mục tiêu

Workshop này hướng dẫn triển khai ứng dụng **AWS Media Vault** trên nền tảng AWS bằng cách sử dụng kiến trúc Serverless Cloud-Native, các dịch vụ được quản lý (Managed Services), xử lý hướng sự kiện (Event-Driven) và cơ chế tải dữ liệu an toàn với S3 Presigned URL. Sau khi hoàn thành workshop, bạn sẽ có thể triển khai một nền tảng lưu trữ và xử lý đa phương tiện hoàn chỉnh với khả năng mở rộng tức thì, tối ưu chi phí (Zero Idle Cost) và đảm bảo an toàn thông tin theo chuẩn AWS Well-Architected.

---

## 1. Giới thiệu bài toán và giải pháp

**AWS Media Vault** là một ứng dụng web cho phép người dùng tải lên, lưu trữ an toàn và tải xuống các tệp tin đa phương tiện (hình ảnh, video). Hệ thống hỗ trợ các chức năng như tải tệp trực tiếp từ trình duyệt, tự động trích xuất thông tin tệp (metadata), ghi nhận nhật ký kiểm toán NoSQL, gửi thông báo tóm tắt tức thì qua email và giám sát sự cố vận hành theo thời gian thực.

Thay vì triển khai ứng dụng trên máy chủ ảo truyền thống (như Amazon EC2) gây lãng phí chi phí nhàn rỗi và đối mặt với rủi ro bảo mật lưu trữ, workshop này áp dụng kiến trúc Serverless hoàn toàn trên AWS. Giao diện người dùng được phân phối tĩnh trực tiếp thông qua **Amazon S3 Static Website Hosting**. Ứng dụng client kết nối an toàn với backend thông qua **Amazon API Gateway** và **AWS Lambda** để tạo **S3 Presigned URL**, cho phép tải tệp trực tiếp lên kho lưu trữ riêng tư **Amazon S3** mà không cần mở public bucket hay để lộ thông tin xác thực AWS.

Để tăng cường tính tự động hóa và khả năng quản lý, metadata của từng tệp tải lên được lưu trữ trên **Amazon DynamoDB**, trong khi báo cáo chi tiết được gửi tự động qua email bằng **Amazon SNS**. Quyền hạn truy cập giữa các dịch vụ được kiểm soát chặt chẽ bằng **AWS IAM** theo nguyên tắc đặc quyền tối thiểu (Least Privilege). Đồng thời, **Amazon CloudWatch** thu thập toàn bộ nhật ký (Logs), số liệu vận hành (Metrics) và kích hoạt cảnh báo (Alarm) khi phát sinh sự cố trong quá trình thực thi.

---

## 2. Kiến trúc hệ thống

Kiến trúc của hệ thống bao gồm các thành phần chính sau:

- Người dùng (Client Web Browser)
- Lưu trữ giao diện tĩnh (Static Web Hosting)
- Cổng kết nối API và kiểm soát CORS (Amazon API Gateway)
- Xử lý logic không máy chủ (AWS Lambda)
- Kho lưu trữ đối tượng riêng tư (Amazon S3 Data Bucket)
- Cơ sở dữ liệu NoSQL (Amazon DynamoDB)
- Hệ thống thông báo đẩy (Amazon SNS)
- Giám sát và cảnh báo thời gian thực (Amazon CloudWatch)

**Hình 1 – Kiến trúc hệ thống AWS Media Vault**

![Kiến trúc hệ thống](/images/5-Workshop/5.1-Workshop-overview/diagram.drawio.png)

---

## 3. Quy trình hoạt động của hệ thống

Luồng xử lý chính của hệ thống diễn ra theo các bước sau:

1. Người dùng truy cập giao diện web portal thông qua endpoint của **Amazon S3 Static Website Hosting**.

2. Khi người dùng chọn tệp cần tải lên, trình duyệt gửi yêu cầu HTTP POST đến **Amazon API Gateway** thông qua endpoint `/media` có bật CORS.

3. API Gateway chuyển tiếp yêu cầu đến **AWS Lambda** bằng cơ chế Lambda Proxy Integration.

4. Hàm Lambda sử dụng AWS SDK (Boto3) để khởi tạo một **S3 Presigned URL (PUT)** có chữ ký số SigV4 với thời hạn 300 giây và trả về cho trình duyệt.

5. Trình duyệt sử dụng Presigned URL để tải trực tiếp tệp tin lên kho lưu trữ riêng tư **Amazon S3 (Data Bucket)** qua phương thức HTTP PUT.

6. Sự kiện `s3:ObjectCreated:*` từ S3 Data Bucket tự động kích hoạt hàm **AWS Lambda** xử lý backend.

7. Lambda trích xuất thông tin tệp (tên tệp, kích thước, định dạng, thời gian tải) và ghi bản ghi vào bảng **Amazon DynamoDB**.

8. Lambda định dạng báo cáo chi tiết và xuất bản thông điệp vào **Amazon SNS Topic**, từ đó SNS tự động gửi email thông báo trạng thái tới người quản trị.

9. Toàn bộ nhật ký hoạt động (Logs) và số liệu (Metrics) được gửi đến **Amazon CloudWatch**. Nếu Lambda phát sinh ngoại lệ, CloudWatch Alarm lập tức chuyển sang trạng thái cảnh báo và thông báo qua SNS.

10. Khi người dùng yêu cầu tải xuống, Lambda khởi tạo **S3 Presigned URL (GET)** có thời hạn để người dùng truy xuất tệp an toàn từ S3.

---

## 4. Các dịch vụ được sử dụng

Workshop sử dụng các dịch vụ AWS sau:

### Giao diện và API

- Amazon S3 (Static Website Hosting)
- Amazon API Gateway (REST API & CORS)

### Dịch vụ tính toán (Compute)

- AWS Lambda (Runtime Python 3.12)

### Lưu trữ & Cơ sở dữ liệu

- Amazon S3 (Data Bucket)
- Amazon DynamoDB

### Thông báo đẩy

- Amazon Simple Notification Service (Amazon SNS)

### Bảo mật & Phân quyền

- AWS Identity and Access Management (AWS IAM)
- S3 Bucket Policy & CORS Configuration

### Giám sát & Vận hành

- Amazon CloudWatch Logs
- Amazon CloudWatch Metrics & Alarms

---

## 5. Kết quả đạt được

Sau khi hoàn thành workshop, bạn sẽ có thể:

- Xây dựng và phân phối một website tĩnh bằng tính năng Amazon S3 Static Website Hosting.
- Thiết lập REST API trên Amazon API Gateway hỗ trợ giao thức CORS cho các phương thức ANY và OPTIONS.
- Lập trình hàm AWS Lambda (Python 3.12) xử lý đa nhiệm giữa tiếp nhận HTTP request và tiêu thụ sự kiện S3.
- Tạo và làm chủ cơ chế chữ ký điện tử S3 Presigned URL (PUT/GET) chuẩn SigV4.
- Cấu hình chia sẻ tài nguyên gốc (CORS) trên Amazon S3 Data Bucket để cho phép upload trực tiếp từ máy khách.
- Lưu trữ và quản lý metadata NoSQL trên Amazon DynamoDB.
- Cấu hình thông báo tức thì qua email với Amazon SNS Topic và Subscription.
- Phân quyền IAM theo nguyên tắc đặc quyền tối thiểu (Least Privilege) và xử lý triệt để lỗi Permissions Boundary.
- Giám sát nhật ký hoạt động và thiết lập CloudWatch Alarm cảnh báo sự cố tự động.
- Thực hiện kiểm thử toàn diện kịch bản tải lên, tải xuống và kiểm thử bơm lỗi (Fault Injection).
- Dọn dẹp sạch sẽ toàn bộ tài nguyên AWS sau khi hoàn thành workshop để tránh phát sinh chi phí.