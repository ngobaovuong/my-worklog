---
title : "Cấu hình Thông báo qua SNS"
date : 2026-09-25
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

### Mục tiêu

Khởi tạo một **Amazon Simple Notification Service (SNS) Topic** theo kiến trúc Publish/Subscribe (Pub/Sub), cấu hình **Email Subscription** để nhận cảnh báo tức thì, tiến hành xác thực liên kết email an toàn và thu thập mã định danh tài nguyên (Topic ARN) để phục vụ cho việc tích hợp vào hàm AWS Lambda và CloudWatch Alarm.

---

## 1. Cơ sở lý thuyết và Mô hình Pub/Sub

Trong kiến trúc đám mây hiện đại, việc tách rời (Decoupling) giữa bộ phận tính toán và bộ phận phát tin là nguyên tắc cốt lõi giúp hệ thống linh hoạt và tin cậy:

- **Mô hình Publisher / Subscriber (Pub/Sub):**
  - **Publisher (Bên phát tin):** Hàm AWS Lambda hoặc CloudWatch Alarm chỉ cần đẩy dữ liệu sự kiện vào một điểm tập kết duy nhất gọi là **Topic**. Publisher không cần biết có bao nhiêu người nhận hay người nhận đang sử dụng giao thức nào (Email, SMS, Webhook hay SQS).
  - **Subscriber (Bên nhận tin):** Các đối tượng đăng ký lắng nghe Topic (ở đây là hòm thư cá nhân của quản trị viên). Ngay khi có thông điệp mới xuất hiện trên Topic, SNS sẽ tự động phân phối (fan-out) thông điệp tới toàn bộ các Subscriber đã xác thực.
- **Phân loại SNS Topic:**
  - **Standard Topic (Được lựa chọn):** Cung cấp thông lượng tin nhắn gần như không giới hạn, đảm bảo chuyển tiếp tin nhắn tốt nhất (Best-effort message delivery) với độ trễ tính bằng mili-giây, hỗ trợ đẩy thông báo qua giao thức Email.
  - **FIFO Topic:** Giữ đúng thứ tự tin nhắn nghiêm ngặt và không trùng lặp, nhưng không hỗ trợ giao thức đầu ra trực tiếp là Email (chỉ hỗ trợ SQS FIFO).
- **Trạng thái Subscription Lifecycle:**
  - Một email khi mới đăng ký vào Topic sẽ nằm ở trạng thái chờ xác thực (**PendingConfirmation**).
  - AWS sẽ gửi một email xác thực chứa token ký điện tử bảo mật. Chỉ khi người sở hữu email nhấp vào liên kết xác nhận, trạng thái mới chuyển sang **Confirmed**, cho phép nhận các bức thư cảnh báo từ hệ thống.

---

## 2. Các bước thực hiện

### Bước 1: Khởi tạo Amazon SNS Topic

1. Đăng nhập vào **AWS Management Console** và đảm bảo Region ở góc trên bên phải là **Asia Pacific (Sydney) ap-southeast-2**.
2. Trên thanh tìm kiếm ở đầu trang, nhập `SNS` và chọn dịch vụ **Simple Notification Service**.
3. Ở menu điều hướng bên trái, nhấp chọn mục **Topics** $\rightarrow$ Nhấn nút màu cam **Create topic**.
![AmazonSNS1](/images/5-Workshop/5.5-SNS-Notification/AmazonSNS1.png)
4. Tại phần **Details**:
   - **Type**: Chọn **Standard**.
   - **Name**: Nhập chính xác tên topic:
     ```text
     MediaProcessingAlerts
     ```
   - **Display name** (tên hiển thị trong tiêu đề email): Nhập:
     ```text
     MediaVault
     ```
5. Các mục **Encryption**, **Access policy**, **Data protection policy**, và **Delivery status logging**: Giữ nguyên toàn bộ cấu hình mặc định.
6. Cuộn xuống dưới cùng và nhấn nút **Create topic**.

```text
Type: Standard
Name: MediaProcessingAlerts
Display name: MediaVault
```

**Checkpoint:** Màn hình chuyển sang trang chi tiết của Topic `MediaProcessingAlerts`. Tại ô **Details**, hãy sao chép lại chuỗi **ARN** để chuẩn bị cho việc tích hợp vào Lambda:
```text
arn:aws:sns:ap-southeast-2:<ACCOUNT-ID>:MediaProcessingAlerts
```

---

### Bước 2: Tạo Subscription nhận thông báo qua Email

1. Vẫn ở trang chi tiết của topic `MediaProcessingAlerts`, cuộn xuống phần danh sách bên dưới và chọn tab **Subscriptions**.
2. Nhấn nút màu cam **Create subscription**.
3. Tại trang cấu hình **Create subscription**:
   - **Topic ARN**: Đã được tự động điền sẵn chuỗi ARN của `MediaProcessingAlerts`.
   - **Protocol**: Nhấp vào menu thả xuống và chọn **Email**.
   - **Endpoint**: Nhập chính xác địa chỉ email cá nhân của bạn (địa chỉ sẽ nhận thông báo khi có file upload hoặc khi hệ thống gặp lỗi).
4. Các mục **Subscription filter policy** và **Redrive policy (dead-letter queue)**: Giữ mặc định.
5. Nhấn nút **Create subscription**.
![AmazonSNS2](/images/5-Workshop/5.5-SNS-Notification/AmazonSNS2.png)
```text
Protocol: Email
Endpoint: your-personal-email@example.com
```

**Checkpoint:** Màn hình quay lại trang chi tiết Topic. Trong bảng **Subscriptions**, bạn sẽ thấy dòng đăng ký email vừa tạo với **Status** hiển thị chữ màu đỏ: `PendingConfirmation`.

---

### Bước 3: Xác thực Email Subscription

1. Mở một tab mới trên trình duyệt và đăng nhập vào hòm thư email bạn vừa nhập ở Bước 2.
2. Kiểm tra hộp thư đến (Inbox) hoặc thư mục Spam/Junk.
3. Tìm bức thư mới nhất gửi từ **AWS Notifications** (`no-reply@sns.amazonaws.com`) với tiêu đề:
   ```text
   AWS Notification - Subscription Confirmation
   ```
4. Mở nội dung email và nhấp vào liên kết **Confirm subscription**.
![AmazonSNS3](/images/5-Workshop/5.5-SNS-Notification/AmazonSNS3.png)
5. Trình duyệt web sẽ mở ra một trang xác thực chính thức từ AWS hiển thị thông báo:
   ```text
   Subscription confirmed!
   You have successfully subscribed to the topic: MediaProcessingAlerts.
   ```
6. Quay lại tab AWS Console trên trình duyệt, nhấn nút biểu tượng làm mới (Refresh) tại bảng Subscriptions.
![AmazonSNS4](/images/5-Workshop/5.5-SNS-Notification/AmazonSNS4.png)
**Checkpoint:** Cột **Status** của email chuyển từ `PendingConfirmation` sang chữ màu xanh lá: **Confirmed**. Mã Subscription ID hợp lệ được hiển thị thay vì trạng thái chờ.

---

## 3. Kết quả mong đợi

- Khởi tạo thành công Standard SNS Topic mang tên `MediaProcessingAlerts` tại Region **ap-southeast-2 (Sydney)**.
- Đăng ký thành công kênh Email Subscription và xác nhận trạng thái **Confirmed** an toàn.
- Thu thập chính xác mã định danh `Topic ARN` để sẵn sàng khai báo vào biến môi trường hoặc mã nguồn Python của AWS Lambda ở chương tiếp theo.
- Hệ thống thông báo đẩy đã sẵn sàng hoạt động để phục vụ cả hai nhiệm vụ: thông báo trạng thái tải tệp thành công và tiếp nhận kích hoạt báo động khẩn cấp từ CloudWatch Alarm.