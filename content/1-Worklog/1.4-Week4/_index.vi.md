---
title: "Worklog Tuần 4"
date: 2026-08-24
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:
* Tìm hiểu tư duy kiến trúc Serverless (Không máy chủ) và dịch vụ điện toán theo sự kiện AWS Lambda.
* Nắm vững vòng đời thực thi của hàm Lambda (Cold Start, Warm Start, Execution Environment).
* Xây dựng mã nguồn xử lý logic dữ liệu bằng Python và Node.js, cấu hình biến môi trường và bộ nhớ (Memory/Timeout).
* Thiết lập các nguồn kích hoạt sự kiện (Event Source Mappings / Triggers) tự động từ Amazon S3 và Amazon API Gateway.
* Theo dõi, debug lỗi runtime và kiểm tra log thực thi thông qua Amazon CloudWatch Logs.

### Các công việc triển khai trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu triết lý Serverless: không cần quản lý máy chủ, tự động co giãn theo lưu lượng từ 0 đến hàng ngàn request, chỉ trả tiền khi code chạy (tính theo mili-giây).<br>- So sánh chi phí và trường hợp sử dụng (Use Cases) giữa EC2, ECS và AWS Lambda.<br>- Phân tích cấu trúc một Function: Handler, Event Object, Context Object. | 24/08/2026 | 24/08/2026 | https://docs.aws.amazon.com/lambda/latest/dg/ |
| 3 | - Khởi tạo hàm Lambda đầu tiên sử dụng runtime Python 3.12.<br>- Cấu hình IAM Execution Role cấp quyền cơ bản `AWSLambdaBasicExecutionRole` để ghi log vào CloudWatch.<br>- Tùy chỉnh thông số phần cứng: điều chỉnh Memory từ 128 MB lên 512 MB và phân tích tác động tương ứng đến vCPU, cấu hình Timeout 15 giây.<br>- Viết mã kiểm thử in thông tin payload và thực hiện test trực tiếp với custom JSON event. | 25/08/2026 | 25/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| 4 | - Xây dựng bài toán xử lý ảnh/tệp tự động: Tạo S3 Bucket tiếp nhận file tải lên.<br>- Cấu hình IAM Role cho phép Lambda có quyền đọc đối tượng `s3:GetObject` từ S3 Bucket.<br>- Thêm S3 Trigger vào Lambda lắng nghe sự kiện `s3:ObjectCreated:*`.<br>- Tải tệp lên S3, kiểm tra Lambda tự động kích hoạt và đọc metadata của tệp. | 26/08/2026 | 26/08/2026 | https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html |
| 5 | - Giới thiệu dịch vụ Amazon API Gateway (REST API và HTTP API).<br>- Tạo một HTTP API làm cổng giao tiếp Webhook tiếp nhận phương thức POST.<br>- Cấu hình Lambda Integration, ánh xạ dữ liệu đầu vào JSON từ HTTP Body vào hàm xử lý.<br>- Sử dụng Postman / cURL để gửi request kiểm thử và nhận response JSON trả về. | 27/08/2026 | 27/08/2026 | https://docs.aws.amazon.com/apigateway/latest/developerguide/ |
| 6 | - Tìm hiểu hiện tượng Cold Start: nguyên nhân, thời gian khởi tạo container và các giải pháp tối ưu hóa (giảm kích thước gói code, sử dụng Provisioned Concurrency).<br>- Phân tích sâu log thực thi trong CloudWatch Log Groups: trường `Duration`, `Billed Duration`, `Memory Size`, `Max Memory Used`.<br>- Tổng hợp kiến thức kiến trúc Serverless, viết tài liệu worklog và hoàn tất mục tiêu tuần 4. | 28/08/2026 | 28/08/2026 | https://cloudjourney.awsstudygroup.com/ |

### Kết quả đạt được tuần 4:
* **Mức độ hoàn thành:** 100%.
* **Kiến thức lý thuyết:**
  * Hiểu rõ mô hình Event-driven Architecture (EDA) và cách các sự kiện gắn kết các microservice phi tập trung.
  * Phân biệt đồng bộ (Synchronous - ví dụ: API Gateway) và bất đồng bộ (Asynchronous - ví dụ: S3, SNS) khi kích hoạt Lambda.
  * Hiểu rõ cơ chế quản lý vòng đời và cơ cấu chi phí tối ưu của Lambda.
* **Kỹ năng thực hành:**
  * Thành thạo viết code xử lý logic bằng Python trên Lambda Handler.
  * Thiết lập luồng xử lý tự động S3 Event Notification -> Lambda Function.
  * Tích hợp thành công API Gateway để tạo ra endpoint API Serverless công khai có khả năng phục vụ hàng ngàn request đồng thời.---
title: "Worklog Tuần 4"
date: 2026-08-24
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:
* Tìm hiểu tư duy kiến trúc Serverless (Không máy chủ) và dịch vụ điện toán theo sự kiện AWS Lambda.
* Nắm vững vòng đời thực thi của hàm Lambda (Cold Start, Warm Start, Execution Environment).
* Xây dựng mã nguồn xử lý logic dữ liệu bằng Python và Node.js, cấu hình biến môi trường và bộ nhớ (Memory/Timeout).
* Thiết lập các nguồn kích hoạt sự kiện (Event Source Mappings / Triggers) tự động từ Amazon S3 và Amazon API Gateway.
* Theo dõi, debug lỗi runtime và kiểm tra log thực thi thông qua Amazon CloudWatch Logs.

### Các công việc triển khai trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu triết lý Serverless: không cần quản lý máy chủ, tự động co giãn theo lưu lượng từ 0 đến hàng ngàn request, chỉ trả tiền khi code chạy (tính theo mili-giây).<br>- So sánh chi phí và trường hợp sử dụng (Use Cases) giữa EC2, ECS và AWS Lambda.<br>- Phân tích cấu trúc một Function: Handler, Event Object, Context Object. | 24/08/2026 | 24/08/2026 | https://docs.aws.amazon.com/lambda/latest/dg/ |
| 3 | - Khởi tạo hàm Lambda đầu tiên sử dụng runtime Python 3.12.<br>- Cấu hình IAM Execution Role cấp quyền cơ bản `AWSLambdaBasicExecutionRole` để ghi log vào CloudWatch.<br>- Tùy chỉnh thông số phần cứng: điều chỉnh Memory từ 128 MB lên 512 MB và phân tích tác động tương ứng đến vCPU, cấu hình Timeout 15 giây.<br>- Viết mã kiểm thử in thông tin payload và thực hiện test trực tiếp với custom JSON event. | 25/08/2026 | 25/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| 4 | - Xây dựng bài toán xử lý ảnh/tệp tự động: Tạo S3 Bucket tiếp nhận file tải lên.<br>- Cấu hình IAM Role cho phép Lambda có quyền đọc đối tượng `s3:GetObject` từ S3 Bucket.<br>- Thêm S3 Trigger vào Lambda lắng nghe sự kiện `s3:ObjectCreated:*`.<br>- Tải tệp lên S3, kiểm tra Lambda tự động kích hoạt và đọc metadata của tệp. | 26/08/2026 | 26/08/2026 | https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html |
| 5 | - Giới thiệu dịch vụ Amazon API Gateway (REST API và HTTP API).<br>- Tạo một HTTP API làm cổng giao tiếp Webhook tiếp nhận phương thức POST.<br>- Cấu hình Lambda Integration, ánh xạ dữ liệu đầu vào JSON từ HTTP Body vào hàm xử lý.<br>- Sử dụng Postman / cURL để gửi request kiểm thử và nhận response JSON trả về. | 27/08/2026 | 27/08/2026 | https://docs.aws.amazon.com/apigateway/latest/developerguide/ |
| 6 | - Tìm hiểu hiện tượng Cold Start: nguyên nhân, thời gian khởi tạo container và các giải pháp tối ưu hóa (giảm kích thước gói code, sử dụng Provisioned Concurrency).<br>- Phân tích sâu log thực thi trong CloudWatch Log Groups: trường `Duration`, `Billed Duration`, `Memory Size`, `Max Memory Used`.<br>- Tổng hợp kiến thức kiến trúc Serverless, viết tài liệu worklog và hoàn tất mục tiêu tuần 4. | 28/08/2026 | 28/08/2026 | https://cloudjourney.awsstudygroup.com/ |

### Kết quả đạt được tuần 4:
* **Mức độ hoàn thành:** 100%.
* **Kiến thức lý thuyết:**
  * Hiểu rõ mô hình Event-driven Architecture (EDA) và cách các sự kiện gắn kết các microservice phi tập trung.
  * Phân biệt đồng bộ (Synchronous - ví dụ: API Gateway) và bất đồng bộ (Asynchronous - ví dụ: S3, SNS) khi kích hoạt Lambda.
  * Hiểu rõ cơ chế quản lý vòng đời và cơ cấu chi phí tối ưu của Lambda.
* **Kỹ năng thực hành:**
  * Thành thạo viết code xử lý logic bằng Python trên Lambda Handler.
  * Thiết lập luồng xử lý tự động S3 Event Notification -> Lambda Function.
  * Tích hợp thành công API Gateway để tạo ra endpoint API Serverless công khai có khả năng phục vụ hàng ngàn request đồng thời.---
title: "Worklog Tuần 4"
date: 2026-08-24
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:
* Tìm hiểu tư duy kiến trúc Serverless (Không máy chủ) và dịch vụ điện toán theo sự kiện AWS Lambda.
* Nắm vững vòng đời thực thi của hàm Lambda (Cold Start, Warm Start, Execution Environment).
* Xây dựng mã nguồn xử lý logic dữ liệu bằng Python và Node.js, cấu hình biến môi trường và bộ nhớ (Memory/Timeout).
* Thiết lập các nguồn kích hoạt sự kiện (Event Source Mappings / Triggers) tự động từ Amazon S3 và Amazon API Gateway.
* Theo dõi, debug lỗi runtime và kiểm tra log thực thi thông qua Amazon CloudWatch Logs.

### Các công việc triển khai trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu triết lý Serverless: không cần quản lý máy chủ, tự động co giãn theo lưu lượng từ 0 đến hàng ngàn request, chỉ trả tiền khi code chạy (tính theo mili-giây).<br>- So sánh chi phí và trường hợp sử dụng (Use Cases) giữa EC2, ECS và AWS Lambda.<br>- Phân tích cấu trúc một Function: Handler, Event Object, Context Object. | 24/08/2026 | 24/08/2026 | https://docs.aws.amazon.com/lambda/latest/dg/ |
| 3 | - Khởi tạo hàm Lambda đầu tiên sử dụng runtime Python 3.12.<br>- Cấu hình IAM Execution Role cấp quyền cơ bản `AWSLambdaBasicExecutionRole` để ghi log vào CloudWatch.<br>- Tùy chỉnh thông số phần cứng: điều chỉnh Memory từ 128 MB lên 512 MB và phân tích tác động tương ứng đến vCPU, cấu hình Timeout 15 giây.<br>- Viết mã kiểm thử in thông tin payload và thực hiện test trực tiếp với custom JSON event. | 25/08/2026 | 25/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| 4 | - Xây dựng bài toán xử lý ảnh/tệp tự động: Tạo S3 Bucket tiếp nhận file tải lên.<br>- Cấu hình IAM Role cho phép Lambda có quyền đọc đối tượng `s3:GetObject` từ S3 Bucket.<br>- Thêm S3 Trigger vào Lambda lắng nghe sự kiện `s3:ObjectCreated:*`.<br>- Tải tệp lên S3, kiểm tra Lambda tự động kích hoạt và đọc metadata của tệp. | 26/08/2026 | 26/08/2026 | https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html |
| 5 | - Giới thiệu dịch vụ Amazon API Gateway (REST API và HTTP API).<br>- Tạo một HTTP API làm cổng giao tiếp Webhook tiếp nhận phương thức POST.<br>- Cấu hình Lambda Integration, ánh xạ dữ liệu đầu vào JSON từ HTTP Body vào hàm xử lý.<br>- Sử dụng Postman / cURL để gửi request kiểm thử và nhận response JSON trả về. | 27/08/2026 | 27/08/2026 | https://docs.aws.amazon.com/apigateway/latest/developerguide/ |
| 6 | - Tìm hiểu hiện tượng Cold Start: nguyên nhân, thời gian khởi tạo container và các giải pháp tối ưu hóa (giảm kích thước gói code, sử dụng Provisioned Concurrency).<br>- Phân tích sâu log thực thi trong CloudWatch Log Groups: trường `Duration`, `Billed Duration`, `Memory Size`, `Max Memory Used`.<br>- Tổng hợp kiến thức kiến trúc Serverless, viết tài liệu worklog và hoàn tất mục tiêu tuần 4. | 28/08/2026 | 28/08/2026 | https://cloudjourney.awsstudygroup.com/ |

### Kết quả đạt được tuần 4:
* **Mức độ hoàn thành:** 100%.
* **Kiến thức lý thuyết:**
  * Hiểu rõ mô hình Event-driven Architecture (EDA) và cách các sự kiện gắn kết các microservice phi tập trung.
  * Phân biệt đồng bộ (Synchronous - ví dụ: API Gateway) và bất đồng bộ (Asynchronous - ví dụ: S3, SNS) khi kích hoạt Lambda.
  * Hiểu rõ cơ chế quản lý vòng đời và cơ cấu chi phí tối ưu của Lambda.
* **Kỹ năng thực hành:**
  * Thành thạo viết code xử lý logic bằng Python trên Lambda Handler.
  * Thiết lập luồng xử lý tự động S3 Event Notification -> Lambda Function.
  * Tích hợp thành công API Gateway để tạo ra endpoint API Serverless công khai có khả năng phục vụ hàng ngàn request đồng thời.---
title: "Worklog Tuần 4"
date: 2026-08-24
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:
* Tìm hiểu tư duy kiến trúc Serverless (Không máy chủ) và dịch vụ điện toán theo sự kiện AWS Lambda.
* Nắm vững vòng đời thực thi của hàm Lambda (Cold Start, Warm Start, Execution Environment).
* Xây dựng mã nguồn xử lý logic dữ liệu bằng Python và Node.js, cấu hình biến môi trường và bộ nhớ (Memory/Timeout).
* Thiết lập các nguồn kích hoạt sự kiện (Event Source Mappings / Triggers) tự động từ Amazon S3 và Amazon API Gateway.
* Theo dõi, debug lỗi runtime và kiểm tra log thực thi thông qua Amazon CloudWatch Logs.

### Các công việc triển khai trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu triết lý Serverless: không cần quản lý máy chủ, tự động co giãn theo lưu lượng từ 0 đến hàng ngàn request, chỉ trả tiền khi code chạy (tính theo mili-giây).<br>- So sánh chi phí và trường hợp sử dụng (Use Cases) giữa EC2, ECS và AWS Lambda.<br>- Phân tích cấu trúc một Function: Handler, Event Object, Context Object. | 24/08/2026 | 24/08/2026 | https://docs.aws.amazon.com/lambda/latest/dg/ |
| 3 | - Khởi tạo hàm Lambda đầu tiên sử dụng runtime Python 3.12.<br>- Cấu hình IAM Execution Role cấp quyền cơ bản `AWSLambdaBasicExecutionRole` để ghi log vào CloudWatch.<br>- Tùy chỉnh thông số phần cứng: điều chỉnh Memory từ 128 MB lên 512 MB và phân tích tác động tương ứng đến vCPU, cấu hình Timeout 15 giây.<br>- Viết mã kiểm thử in thông tin payload và thực hiện test trực tiếp với custom JSON event. | 25/08/2026 | 25/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| 4 | - Xây dựng bài toán xử lý ảnh/tệp tự động: Tạo S3 Bucket tiếp nhận file tải lên.<br>- Cấu hình IAM Role cho phép Lambda có quyền đọc đối tượng `s3:GetObject` từ S3 Bucket.<br>- Thêm S3 Trigger vào Lambda lắng nghe sự kiện `s3:ObjectCreated:*`.<br>- Tải tệp lên S3, kiểm tra Lambda tự động kích hoạt và đọc metadata của tệp. | 26/08/2026 | 26/08/2026 | https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html |
| 5 | - Giới thiệu dịch vụ Amazon API Gateway (REST API và HTTP API).<br>- Tạo một HTTP API làm cổng giao tiếp Webhook tiếp nhận phương thức POST.<br>- Cấu hình Lambda Integration, ánh xạ dữ liệu đầu vào JSON từ HTTP Body vào hàm xử lý.<br>- Sử dụng Postman / cURL để gửi request kiểm thử và nhận response JSON trả về. | 27/08/2026 | 27/08/2026 | https://docs.aws.amazon.com/apigateway/latest/developerguide/ |
| 6 | - Tìm hiểu hiện tượng Cold Start: nguyên nhân, thời gian khởi tạo container và các giải pháp tối ưu hóa (giảm kích thước gói code, sử dụng Provisioned Concurrency).<br>- Phân tích sâu log thực thi trong CloudWatch Log Groups: trường `Duration`, `Billed Duration`, `Memory Size`, `Max Memory Used`.<br>- Tổng hợp kiến thức kiến trúc Serverless, viết tài liệu worklog và hoàn tất mục tiêu tuần 4. | 28/08/2026 | 28/08/2026 | https://cloudjourney.awsstudygroup.com/ |

### Kết quả đạt được tuần 4:
* **Mức độ hoàn thành:** 100%.
* **Kiến thức lý thuyết:**
  * Hiểu rõ mô hình Event-driven Architecture (EDA) và cách các sự kiện gắn kết các microservice phi tập trung.
  * Phân biệt đồng bộ (Synchronous - ví dụ: API Gateway) và bất đồng bộ (Asynchronous - ví dụ: S3, SNS) khi kích hoạt Lambda.
  * Hiểu rõ cơ chế quản lý vòng đời và cơ cấu chi phí tối ưu của Lambda.
* **Kỹ năng thực hành:**
  * Thành thạo viết code xử lý logic bằng Python trên Lambda Handler.
  * Thiết lập luồng xử lý tự động S3 Event Notification -> Lambda Function.
  * Tích hợp thành công API Gateway để tạo ra endpoint API Serverless công khai có khả năng phục vụ hàng ngàn request đồng thời.