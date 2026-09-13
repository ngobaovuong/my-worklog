---
title : "Giám sát & Báo động với CloudWatch"
date : 2026-09-13
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

### Mục tiêu

Thiết lập hệ thống quan sát (Observability) toàn diện theo trụ cột **Operational Excellence** của AWS Well-Architected Framework. Khai thác **Amazon CloudWatch Logs** để theo dõi chi tiết vết thực thi của hàm Lambda, đo lường các chỉ số vận hành (**CloudWatch Metrics**) và cấu hình báo động tự động (**CloudWatch Alarm**) liên kết với Amazon SNS để gửi email cảnh báo khẩn cấp ngay khi phát sinh lỗi hệ thống.

---

## 1. Cơ sở lý thuyết và Kiến trúc Giám sát

Trong các hệ thống Serverless phân tán, việc không thể truy cập trực tiếp vào máy chủ vật lý đòi hỏi phải dựa vào các công cụ thu thập nhật ký và đo lường từ xa:

- **Amazon CloudWatch Logs & Cơ chế Log Group:**
  - Mỗi hàm Lambda khi thực thi sẽ tự động đẩy toàn bộ dữ liệu từ chuẩn `stdout`/`stderr` và các lệnh `print()` trong code về CloudWatch.
  - Toàn bộ log được gom nhóm vào một **Log Group** duy nhất có định dạng chuẩn:
    ```text
    /aws/lambda/process-media-metadata
    ```
  - Bên trong Log Group, mỗi phiên container xử lý (Execution Environment) sẽ tạo ra một **Log Stream** riêng biệt.
  - Cấu trúc bản ghi log chuẩn gồm 3 mốc:
    - `START RequestId`: Bắt đầu phiên thực thi.
    - Application Logs: Các thông tin in ra từ code (ví dụ: `print()`, thông tin lỗi trace, stack trace).
    - `REPORT RequestId`: Tổng kết hiệu năng, gồm `Duration` (thời gian chạy thực tế), `Billed Duration` (thời gian tính tiền), `Memory Size` (dung lượng RAM cấu hình), và `Max Memory Used` (lượng RAM thực tế đã tiêu thụ).
  - *Bài học khắc phục lỗi Log Group không xuất hiện (Log groups: 0):* Nếu hàm Lambda đã chạy nhưng trong CloudWatch hiển thị "There are no log groups", nguyên nhân là do IAM Role bị thiếu quyền tạo log. Việc đính kèm chính sách `AWSLambdaBasicExecutionRole` (cung cấp `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents`) là bắt buộc để Log Group được tự động khởi tạo.

- **Amazon CloudWatch Metrics & Alarms:**
  - AWS Lambda tự động đẩy các chỉ số mặc định về CloudWatch: `Invocations` (số lượt gọi), `Duration` (thời gian chạy), `Errors` (số lượt thực thi thất bại hoặc ném ra Exception chưa được bắt), và `Throttles` (bị giới hạn tần suất gọi).
  - **CloudWatch Alarm (Báo động sự cố):** Theo dõi metric `Errors` của hàm `process-media-metadata`. Khi số lượng lỗi $\ge 1$ trong khoảng thời gian đánh giá (Period 5 phút), Alarm sẽ lập tức chuyển từ trạng thái **OK** sang trạng thái **In alarm** và kích hoạt hành động phát thông báo tới SNS Topic `MediaProcessingAlerts` gửi email khẩn cấp cho quản trị viên.

---

## 2. Các bước thực hiện

### Bước 1: Kiểm tra và Khai thác CloudWatch Logs

1. Đăng nhập vào **AWS Management Console** (đảm bảo đang ở Region **Asia Pacific (Sydney) ap-southeast-2**).
2. Trên thanh tìm kiếm ở đầu trang, gõ `CloudWatch` và chọn dịch vụ **CloudWatch**.
![CloudWatch1](/images/5-Workshop/5.9-Monitoring/CloudWatch1.png)
3. Ở menu điều hướng bên trái, mở rộng mục **Logs** $\rightarrow$ Chọn **Log groups**.
4. Trong ô tìm kiếm log group, nhập:
   ```text
   /aws/lambda/process-media-metadata
   ```
5. Nhấp chuột trực tiếp vào tên Log group vừa tìm thấy.
6. Cuộn xuống phần **Log streams**:
   - Nhấp vào Log stream nằm ở dòng đầu tiên (có mốc thời gian *Last event time* mới nhất).
7. Quan sát và phân tích cấu trúc log:
   - Dòng bắt đầu: `START RequestId: ... Version: $LATEST`
   - Dòng logic thông báo: Các chuỗi in ra từ hàm `lambda_handler`.
   - Dòng kết thúc: `END RequestId: ...`
   - Dòng báo cáo tài nguyên: `REPORT RequestId: ... Duration: 398.17 ms Billed Duration: 944 ms Memory Size: 128 MB Max Memory Used: 100 MB`

```text
Log Group: /aws/lambda/process-media-metadata
Log Status: Streaming operational logs successfully
```

**Checkpoint:** Màn hình hiển thị đầy đủ các dòng log thực thi của Lambda mà không gặp lỗi thiếu quyền truy cập CloudWatch.

---

### Bước 2: Khởi tạo CloudWatch Alarm theo dõi Metric Errors

1. Ở menu bên trái của CloudWatch, mở rộng mục **Alarms** $\rightarrow$ Chọn **All alarms**.
2. Nhấn nút màu cam **Create alarm** ở góc trên bên phải.
3. Tại trang **Specify metric and conditions**, nhấn nút **Select metric**.
![CloudWatch5Alarm](/images/5-Workshop/5.9-Monitoring/CloudWatch5Alarm.png)
4. Màn hình chọn metric hiện ra:
   - Trong thẻ **Browse**, chọn khối dịch vụ **Lambda**.
   - Chọn tiếp danh mục **By Function Name**.
   - Trong ô tìm kiếm, nhập: `process-media-metadata`.
   - Tìm dòng có:
     - **Function Name**: `process-media-metadata`
     - **Metric Name**: `Errors`
   - Tích chọn vào ô vuông ở đầu dòng metric này.
5. Nhấn nút màu cam **Select metric** ở góc dưới cùng bên phải.

```text
Service: Lambda
Category: By Function Name
Function Name: process-media-metadata
Metric Name: Errors
```
![CloudWatch6Alarm](/images/5-Workshop/5.9-Monitoring/CloudWatch6Alarm.png)
![CloudWatch7Alarm](/images/5-Workshop/5.9-Monitoring/CloudWatch7Alarm.png)
![CloudWatch8Alarm](/images/5-Workshop/5.9-Monitoring/CloudWatch8Alarm.png)
**Checkpoint:** Màn hình chuyển sang trang cấu hình chi tiết với biểu đồ hiển thị số lượng lỗi của hàm Lambda.

---

### Bước 3: Thiết lập Điều kiện kích hoạt Báo động (Conditions)

1. Tại phần **Metric**:
   - **Statistic**: Giữ nguyên là **Sum** (Tổng số lỗi).
   - **Period**: Chọn **5 minutes** (khoảng thời gian đánh giá 5 phút).
2. Cuộn xuống phần **Conditions**:
   - **Threshold type**: Chọn **Static**.
   - **Whenever Errors is...**: Chọn **Greater/Equal (>= threshold)**.
   - **than...**: Nhập số:
     ```text
     1
     ```
     *(Ý nghĩa: Chỉ cần phát sinh từ 1 lỗi trở lên trong 5 phút là hệ thống sẽ bật còi báo động).*
3. Mở rộng mục **Additional configuration** (nếu có):
   - **Datapoints to alarm**: Giữ nguyên `1 out of 1`.
   - **Missing data treatment**: Chọn **Treat missing data as good (not breaching threshold)** (xem việc không có dữ liệu là hệ thống đang hoạt động bình thường, không có lỗi).
4. Nhấn nút **Next** ở góc dưới cùng bên phải.

---

### Bước 4: Cấu hình Hành động gửi thông báo qua Amazon SNS

1. Tại trang **Configure actions**:
2. Trong mục **Alarm state trigger**: Chọn trạng thái **In alarm** (Kích hoạt khi báo động phát nổ).
3. Trong mục **Send a notification to the following SNS topic**:
   - Chọn **Select an existing SNS topic**.
   - Tại ô **Send a notification to...**: Bấm vào và chọn đúng topic bạn đã tạo ở Mục 5.5:
     ```text
     MediaProcessingAlerts
     ```
   - Quan sát bên dưới: Hệ thống hiển thị địa chỉ email cá nhân của bạn đã được xác nhận (Confirmed).
4. Nhấn nút **Next** ở góc dưới bên phải.

```text
Alarm state trigger: In alarm
Notification action: Send a notification to an existing SNS topic
SNS Topic: MediaProcessingAlerts
```

---

### Bước 5: Đặt tên và Hoàn tất tạo Alarm

1. Tại trang **Add name and description**:
   - **Alarm name**: Nhập chính xác tên sau:
     ```text
     MediaLambdaFailureAlarm
     ```
   - **Alarm description**: Nhập:
     ```text
     Canh bao tu dong khi ham Lambda process-media-metadata gap su co thuc thi
     ```
2. Nhấn nút **Next**.
3. Tại trang **Preview and create**:
   - Xem lại toàn bộ biểu đồ, điều kiện ngưỡng (`Errors >= 1 trong 5 phút`) và hành động gửi thông báo qua SNS.
4. Cuộn xuống cuối trang và nhấn nút màu cam **Create alarm**.

**Checkpoint:** Bảng danh sách Alarms xuất hiện dòng `MediaLambdaFailureAlarm`. 
- Trong 1-2 phút đầu tiên, trạng thái có thể hiển thị là **Insufficient data** (Đang chờ thu thập dữ liệu).
- Sau khoảng vài phút khi hệ thống không có lỗi nào, trạng thái sẽ tự động chuyển sang màu xanh lá **OK**.

---

## 3. Kết quả mong đợi

- Log Group `/aws/lambda/process-media-metadata` được cấu hình và thu thập đầy đủ nhật ký thời gian thực từ hàm Lambda.
- Tạo thành công CloudWatch Alarm `MediaLambdaFailureAlarm` theo dõi metric `Errors` của Lambda với ngưỡng kích hoạt $\ge 1$.
- Báo động được liên kết trực tiếp với Amazon SNS Topic `MediaProcessingAlerts`, đảm bảo email cảnh báo sẽ được gửi tự động tới quản trị viên khi hệ thống gặp sự cố.
- Toàn bộ cơ chế giám sát và cảnh báo đã sẵn sàng cho bước kiểm thử hệ thống và kiểm thử bơm lỗi (Fault Injection) ở Mục 5.10.