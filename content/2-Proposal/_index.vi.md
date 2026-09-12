---
title: "Đề xuất"
date: 2026-09-25
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AWS Media Vault

## Hệ thống Lưu trữ Đa phương tiện & Giám sát Serverless trên AWS

---

# 1. Tóm tắt

AWS Media Vault là một giải pháp đám mây không máy chủ (Serverless) cho phép người dùng tải lên, lưu trữ và truy xuất tệp tin đa phương tiện (ảnh, video) một cách an toàn và tự động. Nền tảng cung cấp giao diện web portal trực quan, tự động trích xuất thông tin tệp, lưu trữ dữ liệu kiểm toán, phát thông báo tức thì qua email và giám sát lỗi vận hành thời gian thực, đồng thời sử dụng hoàn toàn các dịch vụ AWS được quản lý để tối ưu chi phí, đảm bảo tính sẵn sàng cao và loại bỏ hoàn toàn gánh nặng bảo trì máy chủ.

Ứng dụng phía máy khách được xây dựng bằng **HTML5**, **JavaScript (ES6+)** và **CSS3**, được lưu trữ tĩnh trực tiếp trên **Amazon S3 (Static Website Hosting)**. Giao diện kết nối an toàn với hạ tầng đám mây thông qua **Amazon API Gateway** và **AWS Lambda** bằng cơ chế **S3 Presigned URL**, giúp người dùng tải dữ liệu trực tiếp lên kho lưu trữ riêng tư mà không cần mở public bucket hay để lộ thông tin xác thực AWS.

Hạ tầng backend sử dụng **AWS Lambda** (Python 3.12) làm trung tâm xử lý hướng sự kiện (Event-driven). Metadata của từng tệp được đồng bộ hóa tức thì vào **Amazon DynamoDB**, thông báo chi tiết về dung lượng, định dạng và thời gian tải lên được gửi đến người quản trị qua **Amazon SNS**, các số liệu thực thi cùng nhật ký hệ thống được ghi nhận bởi **Amazon CloudWatch**, và toàn bộ quyền hạn được thắt chặt bằng **AWS IAM** theo nguyên tắc đặc quyền tối thiểu (Least Privilege). Kiến trúc này cung cấp giải pháp xử lý dữ liệu tự động, bảo mật cao, chi phí gần như bằng 0 và sẵn sàng mở rộng quy mô tức thì.

---

# 2. Vấn đề

## Vấn đề hiện tại

Các hệ thống lưu trữ và xử lý tệp truyền thống thường triển khai trên máy chủ ảo (như Amazon EC2 hoặc VPS truyền thống), gây ra nhiều hạn chế lớn:

- **Lãng phí chi phí nhàn rỗi (Idle Cost):** Máy chủ vẫn phải chạy và tính tiền 24/7 ngay cả khi không có người dùng tải file lên.
- **Rủi ro bảo mật lưu trữ:** Việc cấp quyền upload thường dẫn đến việc mở public bucket hoặc nhúng trực tiếp AWS Access Key/Secret Key vào mã nguồn frontend.
- **Gánh nặng nghẽn cổ chai (Bottleneck):** Máy chủ web phải trung chuyển toàn bộ dữ liệu tệp nặng (video, ảnh chất lượng cao) trước khi đẩy vào kho lưu trữ, gây quá tải RAM và CPU.
- **Thiếu khả năng phản ứng thời gian thực:** Quản trị viên không nhận được cảnh báo ngay lập tức khi xuất hiện tệp mới hoặc khi tiến trình xử lý gặp sự cố gián đoạn.

## Giải pháp

Giải pháp đề xuất là xây dựng kiến trúc Serverless Event-Driven hoàn chỉnh dựa trên các dịch vụ AWS được quản lý:

- Người dùng truy cập cổng thông tin được phân phối trực tiếp từ **Amazon S3 Website Hosting**, yêu cầu URL tải lên/tải xuống an toàn.
- **Amazon API Gateway** tiếp nhận request và kích hoạt **AWS Lambda** tạo mã xác thực có thời hạn (**S3 Presigned URL**). Người dùng thực hiện upload trực tiếp lên **Amazon S3 Data Bucket**.
- Sự kiện `s3:ObjectCreated:*` tự động kích hoạt Lambda để chuẩn hóa thông tin, ghi nhận nhật ký kiểm toán vào **Amazon DynamoDB** và gửi email tóm tắt định dạng chuẩn qua **Amazon SNS**.
- Mọi lỗi thực thi được **Amazon CloudWatch Metric Filters & Alarms** bắt giữ và cảnh báo tự động.

## Lợi ích

Kiến trúc đề xuất mang lại các lợi ích vượt trội:

- **Chi phí tối ưu (Zero Idle Cost):** Chỉ phát sinh chi phí tính toán theo từng mili-giây khi có request thực tế, nằm trọn trong AWS Free Tier.
- **Bảo mật tuyệt đối:** S3 Data Bucket chặn toàn bộ truy cập public; trao đổi tệp chỉ diễn ra qua chữ ký điện tử tạm thời (Presigned URL).
- **Khả năng mở rộng không giới hạn:** Kiến trúc serverless tự động co giãn tức thì từ vài tệp đến hàng triệu tệp tải lên cùng lúc.
- **Vận hành tự động:** Loại bỏ hoàn toàn công việc vá lỗi hệ điều hành, cấu hình mạng phức tạp hay quản lý cụm máy chủ.
- **Giám sát minh bạch:** Ghi log chi tiết và cảnh báo thời gian thực về email người quản trị.

---

# 3. Kiến trúc giải pháp

Hệ thống ứng dụng mô hình kiến trúc Event-Driven Serverless, tách biệt hoàn toàn giữa tầng lưu trữ frontend và tầng xử lý dữ liệu backend.

## Kiến trúc giải pháp

![Kiến trúc hệ thống](/images/2-Proposal/system_architecture.png)

## Các dịch vụ AWS sử dụng

- **Amazon S3 (Hosting Bucket):** Lưu trữ mã nguồn web tĩnh và phục vụ giao diện người dùng.
- **Amazon S3 (Data Bucket):** Kho lưu trữ tệp tin ảnh và video riêng tư.
- **Amazon API Gateway:** Cung cấp REST API endpoint kết nối bảo mật giữa Frontend và Lambda.
- **AWS Lambda:** Hàm tính toán không máy chủ xử lý cấp phép Presigned URL và phân tích sự kiện S3.
- **Amazon DynamoDB:** Cơ sở dữ liệu NoSQL lưu trữ thông tin kiểm toán và metadata của tệp.
- **Amazon SNS (Simple Notification Service):** Dịch vụ gửi thông báo email tự động cho người quản trị.
- **Amazon CloudWatch:** Dịch vụ thu thập Logs, theo dõi Metrics và kích hoạt Alarm khi phát sinh lỗi.
- **AWS IAM:** Quản trị định danh và chính sách phân quyền tối thiểu (Least Privilege).

## Thiết kế thành phần

### Frontend

- HTML5
- CSS3
- JavaScript (Fetch API, ES6+)
- Lưu trữ trên S3 Static Website Hosting

### Backend & API

- Amazon API Gateway (REST API, CORS Enabled)
- AWS Lambda (Runtime Python 3.12)
- AWS SDK for Python (Boto3)

### Cơ sở dữ liệu

- Amazon DynamoDB (On-Demand / Provisioned Capacity)

### Lưu trữ tệp tin

- Amazon S3 Standard (CORS Enabled, S3v4 Signature)

### Giám sát & Cảnh báo

- Amazon SNS Topic (Email Protocol)
- Amazon CloudWatch Log Groups
- Amazon CloudWatch Alarms (Metric `Errors` $\ge 1$)

### Luồng dữ liệu (Workflow)

Trình duyệt Web

↓ *(1. Xin Presigned URL qua REST API)*

Amazon API Gateway

↓ *(2. Gọi hàm sinh URL tạm)*

AWS Lambda

↓ *(3. Upload file trực tiếp bằng HTTP PUT)*

Amazon S3 (Data Bucket)

↓ *(4. Kích hoạt sự kiện s3:ObjectCreated)*

AWS Lambda

├── *(5a. Ghi Metadata)* ──> Amazon DynamoDB

├── *(5b. Báo cáo chi tiết)* ──> Amazon SNS ──> Quản trị viên (Email)

└── *(5c. Ghi nhận Logs & Metrics)* ──> Amazon CloudWatch

---

# 4. Triển khai kỹ thuật

## Các giai đoạn triển khai

Dự án được triển khai qua các giai đoạn chi tiết sau:

- Nghiên cứu mô hình kiến trúc Event-Driven Serverless và các tiêu chuẩn bảo mật AWS Well-Architected.
- Thiết kế sơ đồ kiến trúc hệ thống và luồng dữ liệu end-to-end.
- Thiết lập hạ tầng lưu trữ S3 Data Bucket và cấu hình chia sẻ tài nguyên gốc (CORS).
- Xây dựng bảng cơ sở dữ liệu NoSQL trên Amazon DynamoDB.
- Cấu hình chủ đề Amazon SNS Topic và xác thực đăng ký nhận email (Subscription).
- Thiết kế IAM Role và kiểm soát chính sách phân quyền tối thiểu (Least Privilege).
- Phát triển mã nguồn AWS Lambda (Python 3.12) tích hợp Boto3 xử lý đa nhiệm (API + Event).
- Khởi tạo Amazon API Gateway REST API, cấu hình CORS và triển khai stage `prod`.
- Xây dựng giao diện web portal tĩnh bằng HTML/JS và cấu hình S3 Static Website Hosting.
- Thiết lập giám sát CloudWatch Log Groups và cấu hình CloudWatch Alarm khi có lỗi phát sinh.
- Thực hiện kiểm thử toàn diện kịch bản tải lên, tải xuống, kiểm tra log và kiểm thử bơm lỗi (Fault Injection).
- Tối ưu hóa tài nguyên và lập quy trình dọn dẹp (Clean-up).

## Yêu cầu kỹ thuật

### Ngôn ngữ lập trình

- Python 3.12
- JavaScript (Vanilla ES6)
- HTML5 / CSS3

### Thư viện & Công cụ cốt lõi

- Boto3 (AWS SDK for Python)
- Botocore (Config Signature Version S3v4)
- Postman / cURL (Kiểm thử API)

### Dịch vụ đám mây (AWS)

- Amazon S3
- Amazon API Gateway
- AWS Lambda
- Amazon DynamoDB
- Amazon SNS
- Amazon CloudWatch
- AWS IAM

### Công cụ phát triển

- Visual Studio Code
- Git & GitHub
- AWS Management Console
- Draw.io / Excalidraw

---

# 5. Lộ trình & Các mốc

Dự án được hoàn thành qua các giai đoạn cụ thể sau:

### Giai đoạn 1 – Thiết kế & Chuẩn bị

- Phân tích yêu cầu bài toán xử lý dữ liệu phương tiện Serverless.
- Vẽ sơ đồ kiến trúc và luồng dữ liệu end-to-end.
- Xác định ma trận phân quyền IAM cho các dịch vụ.

### Giai đoạn 2 – Thiết lập tài nguyên lõi Backend

- Tạo S3 Data Bucket và cấu hình CORS.
- Khởi tạo bảng `MediaMetadata` trên DynamoDB với khóa chính `FileId`.
- Thiết lập SNS Topic `MediaProcessingAlerts` và xác thực email quản trị.

### Giai đoạn 3 – Phát triển Logic tính toán (Compute)

- Thiết lập IAM Execution Role loại bỏ Permissions Boundary không phù hợp.
- Phát triển mã nguồn hàm Lambda xử lý linh hoạt 2 tác vụ: cấp Presigned URL và xử lý sự kiện S3.
- Kiểm thử đơn vị (Unit Test) trực tiếp trên Lambda console bằng mock event `s3-put`.

### Giai đoạn 4 – Tích hợp API & Phát triển Web Portal

- Dựng REST API Gateway với resource `/media` và method `ANY`.
- Bật cấu hình CORS cho API Gateway và triển khai (Deploy) stage `prod`.
- Viết mã nguồn frontend `index.html` tích hợp Fetch API gọi REST endpoint.
- Triển khai hosting giao diện web tĩnh qua S3 Static Website Hosting.

### Giai đoạn 5 – Giám sát, Cảnh báo & Tối ưu

- Gán chính sách `AWSLambdaBasicExecutionRole` để tự động hóa tạo CloudWatch Log Group.
- Thiết lập CloudWatch Alarm giám sát metric `Errors` của Lambda với ngưỡng kích hoạt $\ge 1$.
- Tích hợp liên kết Alarm trực tiếp vào SNS Topic để cảnh báo qua email.

### Giai đoạn 6 – Kiểm thử hệ thống & Bơm lỗi

- Kiểm thử kịch bản thành công: Tải file từ giao diện web $\rightarrow$ kiểm tra dữ liệu trong S3, DynamoDB và email từ SNS.
- Kiểm thử kịch bản lỗi: Cố tình thay đổi phân quyền IAM để xác thực cơ chế ghi log và kích hoạt báo động CloudWatch Alarm về email.
- Kiểm tra tính năng tạo Presigned URL tải xuống (Download) có thời hạn 5 phút.

### Giai đoạn 7 – Đóng gói & Báo cáo

- Viết tài liệu hướng dẫn thực hành (Workshop) song ngữ VI/EN.
- Đóng gói mã nguồn lên GitHub repository.
- Chuẩn bị slide và báo cáo nghiệm thu dự án.

---

# 6. Ước tính chi phí

## Ước tính chi phí hạ tầng

Toàn bộ giải pháp được thiết kế tối ưu trên nền tảng Serverless, tận dụng tối đa gói **AWS Free Tier**:

| Dịch vụ AWS | Hạn mức miễn phí (Free Tier) | Mức sử dụng dự kiến | Chi phí ước tính |
|---|---|---|---|
| **Amazon S3** | 5 GB Standard Storage, 20.000 GET, 2.000 PUT | ~200 MB, ~500 requests | 0.00 USD/tháng |
| **AWS Lambda** | 1.000.000 requests/tháng, 3.200.000 giây tính toán | ~1.000 invocations | 0.00 USD/tháng |
| **Amazon API Gateway** | 1.000.000 API calls/tháng (12 tháng đầu) | ~1.500 calls | 0.00 USD/tháng |
| **Amazon DynamoDB** | 25 GB dữ liệu, 25 WCU / 25 RCU miễn phí | < 10 MB, 5 WCU / 5 RCU | 0.00 USD/tháng |
| **Amazon SNS** | 1.000 thông báo email miễn phí mỗi tháng | ~100 emails | 0.00 USD/tháng |
| **Amazon CloudWatch** | 10 Metrics tùy chỉnh, 10 Alarms, 5 GB Logs | 1 Alarm, 1 Log Group (~50 MB) | 0.00 USD/tháng |
| **AWS IAM** | Miễn phí hoàn toàn | Không giới hạn Policy/Role | 0.00 USD/tháng |
| **Tổng ước tính** | | | **~0.00 USD/tháng** |

### Hướng dẫn kiểm soát chi phí

- **AWS Budgets:** Thiết lập ngân sách cảnh báo tự động gửi email khi tổng chi phí vượt quá **1.00 USD**.
- **Cơ chế Presigned URL:** Giới hạn thời gian hiệu lực của đường link tải lên/tải xuống tối đa **300 giây (5 phút)** để tránh lạm dụng băng thông.
- **S3 Lifecycle Rules:** Thiết lập quy tắc chuyển đổi tệp sang *S3 Standard-IA* hoặc tự động xóa tệp thử nghiệm sau 7 ngày.
- **Quy trình dọn dẹp sau nghiệm thu:** Thực hiện dọn dẹp sạch sẽ tài nguyên theo hướng dẫn: làm rỗng (Empty) và xóa cả 2 S3 Buckets, xóa API Gateway, Lambda Function, DynamoDB Table, CloudWatch Alarm/Logs và SNS Topic để tránh phát sinh chi phí tiềm ẩn sau thời gian Free Tier.

---

# 7. Đánh giá rủi ro

## Ma trận rủi ro

- Lỗi phân quyền IAM (Access Denied) giữa Lambda và DynamoDB/SNS do Permissions Boundary.
- Lỗi CORS (Cross-Origin Resource Sharing) khi trình duyệt gửi request qua API Gateway hoặc upload S3.
- Vòng lặp vô hạn (Infinite Loop) nếu Lambda xử lý lại ghi file vào cùng prefix kích hoạt sự kiện S3.
- Lộ lọt thông tin bảo mật đám mây (AWS Credentials) trên mã nguồn frontend.
- Thiếu hụt nhật ký giám sát do thiếu quyền ghi CloudWatch Logs.
- Phát sinh chi phí ngoài ý muốn do tải lên tệp dung lượng quá lớn hoặc bị spam API.

## Chiến lược giảm thiểu

- Rà soát ma trận IAM, loại bỏ Permissions Boundary không cần thiết và áp dụng chặt chẽ Least Privilege.
- Cấu hình chi tiết CORS Headers trên cả S3 Bucket và API Gateway (xử lý triệt để method OPTIONS preflight).
- Cô lập S3 Hosting Bucket và S3 Data Bucket riêng biệt; không cấu hình Lambda ghi đè vào source prefix.
- Sử dụng mô hình Presigned URL tạo từ backend thay vì truyền Access Key về phía máy khách.
- Đính kèm chính sách được quản lý `AWSLambdaBasicExecutionRole` cho mọi Lambda Role.
- Đặt giới hạn kích thước tệp tải lên và thiết lập CloudWatch Billing Alarm.

## Kế hoạch dự phòng

- **Khi gặp sự cố phân quyền:** Kiểm tra CloudWatch Log Stream hoặc sử dụng tính năng Test trực tiếp trên Lambda console để đọc chính xác thông báo lỗi `AccessDeniedException`.
- **Khi API Gateway gián đoạn:** Kích hoạt tính năng kiểm thử cURL nội bộ để phân tách lỗi do frontend hay backend.
- **Khi CloudWatch Alarm kích hoạt:** Quản trị viên truy cập ngay log group `/aws/lambda/process-media-metadata` để kiểm tra stack trace và khắc phục lỗi logic code.
- **Khi tệp lỗi định dạng:** Bổ sung khối bắt lỗi `try...except` và định tuyến thông báo lỗi chi tiết về hòm thư thông qua kênh riêng biệt.

---

# 8. Kết quả mong đợi

## Kết quả kỹ thuật

Dự án hoàn thành cung cấp:

- Một ứng dụng web portal hoàn chỉnh hỗ trợ tải lên và tải xuống tệp tin phương tiện đa định dạng.
- Kiến trúc xử lý hướng sự kiện tự động 100% không cần quản trị máy chủ (Zero Server Management).
- Cơ chế bảo mật S3 Presigned URL được ký bằng thuật toán SigV4 an toàn, phân quyền tối thiểu với IAM.
- Hệ thống quản lý thông tin tệp tập trung trên cơ sở dữ liệu NoSQL DynamoDB.
- Pipeline cảnh báo thông minh qua email (Amazon SNS) với đầy đủ thông tin metadata chi tiết.
- Hệ thống giám sát vận hành thời gian thực kết hợp bắt lỗi tự động với Amazon CloudWatch Alarms.
- Tài liệu hướng dẫn workshop kỹ thuật song ngữ chi tiết, có khả năng tái thực hiện độc lập.

## Giá trị kinh doanh

Dự án minh chứng rõ nét khả năng hiện thực hóa các nguyên tắc thuộc AWS Well-Architected Framework:

- **Tối ưu chi phí:** Giảm thiểu 100% chi phí máy chủ nhàn rỗi so với mô hình EC2 truyền thống.
- **Vận hành xuất sắc:** Toàn bộ chu trình từ tiếp nhận, xử lý đến cảnh báo được tự động hóa end-to-end.
- **Bảo mật & Độ tin cậy:** Bảo vệ dữ liệu người dùng ở mức cao nhất, đảm bảo tính toàn vẹn và độ bền dữ liệu đạt mức 99.999999999% (11 số 9) của Amazon S3.

Các hướng phát triển mở rộng trong tương lai có thể bao gồm tích hợp **AWS Rekognition** để tự động nhận diện khuôn mặt và gắn nhãn nội dung ảnh, thêm **Amazon CloudFront** tăng tốc phân phối nội dung toàn cầu và tích hợp **Amazon Cognito** để phân quyền người dùng đa cấp độ.