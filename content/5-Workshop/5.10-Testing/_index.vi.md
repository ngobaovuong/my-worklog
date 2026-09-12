---
title : "Kiểm thử hệ thống & Bơm lỗi"
date : 2026-09-25
weight : 10
chapter : false
pre : " <b> 5.10. </b> "
---

### Mục tiêu

Thực hiện kiểm thử tích hợp toàn diện (End-to-End Testing) cho nền tảng **AWS Media Vault** để xác nhận tính toàn vẹn của luồng dữ liệu từ giao diện web người dùng đến các dịch vụ hạ tầng AWS. Đồng thời thực hiện kỹ thuật **Bơm lỗi thực tế (Fault Injection)** nhằm xác thực cơ chế ghi nhật ký chi tiết trên Amazon CloudWatch và kiểm chứng khả năng tự động kích hoạt chuông báo động (Alarm) gửi email khẩn cấp qua Amazon SNS.

---

## 1. Cơ sở lý thuyết và Kịch bản kiểm thử

Để đánh giá một hệ thống Cloud-Native đạt chuẩn AWS Well-Architected, quy trình kiểm thử cần bao gồm cả hai trường hợp:

### 1. Kịch bản vận hành tiêu chuẩn (Happy Path Validation):
- **Tính năng Tải lên (Upload Pipeline):**
  1. Trình duyệt client gửi HTTP POST tới API Gateway `/media` xin cấp quyền S3 Presigned URL.
  2. API Gateway chuyển tiếp tới Lambda, Lambda dùng Boto3 tạo chữ ký số SigV4 với thời hạn 300 giây.
  3. Client dùng Presigned URL để đẩy trực tiếp file nhị phân (Binary stream) lên S3 Data Bucket `fcaj-media-source-demo-2026` qua HTTP `PUT`.
  4. S3 Data Bucket phát sinh sự kiện `s3:ObjectCreated:*` kích hoạt Lambda.
  5. Lambda trích xuất metadata, bóc tách định dạng, tính toán dung lượng theo KB, giải mã URL an toàn bằng `urllib.parse.unquote_plus`.
  6. Lambda ghi bản ghi vào bảng DynamoDB `MediaMetadata`.
  7. Lambda xuất bản thông điệp vào SNS Topic `MediaProcessingAlerts` gửi email báo cáo chi tiết đến hộp thư quản trị viên.
- **Tính năng Tải xuống an toàn (Secure Download):**
  1. Client gửi tên file cần tải về tới API Gateway.
  2. Lambda sinh Presigned URL `GET` có thời hạn 5 phút.
  3. Client nhấp vào liên kết để tải file trực tiếp từ S3 về máy tính mà không cần quyền public trên bucket.

### 2. Kịch bản Bơm lỗi thực tế (Fault Injection & Incident Response):
- Mô phỏng sự cố thực tế: Cố tình thay đổi quyền hạn của IAM Role bằng cách tước quyền `dynamodb:PutItem` (tái hiện lỗi `AccessDeniedException` hoặc xung đột Permissions Boundary đã phân tích ở Mục 5.3).
- **Kiểm chứng phản ứng hệ thống:**
  - Client vẫn upload được file lên S3, nhưng tiến trình xử lý ngầm của Lambda gặp lỗi nghiêm trọng (Unhandled ClientError Exception).
  - CloudWatch Logs bắt trọn vẹn Stack Trace và mã lỗi của SDK Boto3.
  - Metric `Errors` của Lambda tăng vọt lên $\ge 1$.
  - CloudWatch Alarm `MediaLambdaFailureAlarm` lập tức chuyển trạng thái sang **In alarm** màu đỏ.
  - Amazon SNS tự động phát thông báo khẩn cấp tới email cá nhân với tiêu đề sự cố từ AWS.

---

## 2. Các bước thực hiện

### Kịch bản 1: Kiểm thử luồng vận hành tiêu chuẩn (Happy Path)

#### Bước 1: Mở giao diện Web Portal từ S3 Static Website Endpoint
1. Mở trình duyệt web (Google Chrome hoặc Firefox).
2. Dán đường dẫn **Bucket website endpoint** đã lấy ở Mục 5.8:
   ```text
   [http://fcaj-media-portal-web-2026.s3-website-ap-southeast-2.amazonaws.com](http://fcaj-media-portal-web-2026.s3-website-ap-southeast-2.amazonaws.com)
   ```
3. Nhấn phím **F12** trên bàn phím để mở thanh công cụ **Developer Tools** $\rightarrow$ Chuyển sang tab **Console** và **Network** (để theo dõi các gói tin API).
![Test1](/images/5-Workshop/5.10-Testing/Test1.png)
#### Bước 2: Thực hiện tải lên tệp tin (Upload)
1. Tại phần **1. Tải lên Ảnh / Video (Upload)**, nhấn nút **Choose File** (hoặc *Chọn tệp*).
2. Chọn một tệp hình ảnh hoặc video bất kỳ trên máy tính (ví dụ: `meomeo.jfif` hoặc `demo.png`).
3. Nhấn nút màu cam **Tải lên S3 Cloud**.
4. Quan sát phản hồi trên giao diện web:
   - Thông báo chuyển từ màu xanh dương (*Đang yêu cầu cấp quyền...*) sang màu xanh lá cây:
     ```text
     Tải lên thành công!
     • Tên tệp: meomeo.jfif
     • Kích thước: 23.50 KB
     • S3 Trigger: Đã tự động kích hoạt Lambda trích xuất Metadata vào DynamoDB.
     • Thông báo: Vui lòng kiểm tra hộp thư email cá nhân để xem bản tin chi tiết từ Amazon SNS.
     ```
![Test2](/images/5-Workshop/5.10-Testing/Test2.png)
**Checkpoint:** Màn hình Web Portal hiển thị thông báo thành công màu xanh lá, không xuất hiện bất kỳ lỗi đỏ nào trong tab Console của Developer Tools.

#### Bước 3: Xác minh dữ liệu trên Amazon S3 Console
1. Đăng nhập AWS Console $\rightarrow$ Mở dịch vụ **Amazon S3**.
2. Nhấp vào bucket **`fcaj-media-source-demo-2026`**.
3. Tại tab **Objects**, kiểm tra danh sách tệp:
   - Tệp `meomeo.jfif` đã xuất hiện với đúng kích thước và thời gian tải lên.
![Test3](/images/5-Workshop/5.10-Testing/Test3.png)
#### Bước 4: Xác minh bản ghi Metadata trên Amazon DynamoDB
1. Mở dịch vụ **Amazon DynamoDB** $\rightarrow$ Chọn mục **Tables** ở cột bên trái.
2. Nhấp vào tên bảng **`MediaMetadata`**.
3. Nhấn nút **Explore table items** ở góc trên bên phải.
4. Trong bảng **Items returned**, nhấp vào bản ghi mới nhất và kiểm tra các trường dữ liệu:
   - `FileId`: `meomeo.jfif`
   - `UploadTime`: Mốc thời gian chuẩn UTC (ví dụ: `2026-09-11 06:45:20 UTC`)
   - `Bucket`: `fcaj-media-source-demo-2026`
   - `Format`: `JFIF`
   - `SizeBytes`: `24064`
   - `Status`: `ACTIVE`
![Test4](/images/5-Workshop/5.10-Testing/Test4.png)
![Test5](/images/5-Workshop/5.10-Testing/Test5.png)
#### Bước 5: Xác minh Email thông báo từ Amazon SNS
1. Mở hòm thư email cá nhân của bạn.
2. Kiểm tra hộp thư đến (Inbox) hoặc mục Thư rác (Spam).
3. Mở bức thư gửi từ **AWS Notifications** với tiêu đề:
   ```text
   [AWS Media Vault] File meomeo.jfif Processed Successfully
   ```
4. Nội dung thư hiển thị bảng báo cáo chi tiết:
   ```text
   ===========================================
          AWS MEDIA VAULT - NEW UPLOAD        
   ===========================================
   • Tên tệp tin    : meomeo.jfif
   • Định dạng      : JFIF
   • Dung lượng     : 24064 Bytes (~23.5 KB)
   • Kho lưu trữ    : fcaj-media-source-demo-2026
   • Thời gian tải  : 2026-09-11 06:45:20 UTC
   • Cơ sở dữ liệu  : DynamoDB [MediaMetadata] - SUCCESS
   ===========================================
   Trạng thái: Tệp đã lưu trữ thành công và sẵn sàng truy xuất an toàn.
   ```
![Test6](/images/5-Workshop/5.10-Testing/Test6.png)
#### Bước 6: Kiểm tra tính năng Tải xuống tệp an toàn (Download)
1. Quay lại trang Web Portal trên trình duyệt.
2. Tại phần **2. Tải xuống tệp an toàn (Download)**:
   - Nhập chính xác tên tệp vừa tải lên: `meomeo.jfif`.
3. Nhấn nút màu xanh đen **Tạo liên kết tải an toàn**.
4. Giao diện xuất hiện khung màu xanh lá chứa đường link:
   ```text
   [ Bấm vào đây để tải xuống: meomeo.jfif ]
   ```
5. Nhấp chuột vào đường link này: Trình duyệt tự động mở hoặc tải file ảnh `meomeo.jfif` về máy tính thành công thông qua chữ ký số tạm thời SigV4.
![Test7](/images/5-Workshop/5.10-Testing/Test7.png)
**Checkpoint:** Tải xuống tệp thành công, hoàn tất kiểm thử end-to-end cho Happy Path.

---

### Kịch bản 2: Kiểm thử Bơm lỗi & Kích hoạt Báo động (Fault Injection & Alarm Testing)

#### Bước 1: Cố tình phá vỡ phân quyền IAM (Tước quyền DynamoDB)
1. Trên AWS Console, mở dịch vụ **IAM** $\rightarrow$ Chọn mục **Roles**.
2. Nhấp vào role **`LambdaMediaProcessingRole`**.
3. Tại tab **Permissions**, tìm policy **`LambdaMediaPipelinePolicy`** $\rightarrow$ Nhấn nút mũi tên mở rộng $\rightarrow$ Nhấn nút **Edit**.
4. Chuyển sang tab **JSON**, sửa dòng action của DynamoDB từ:
   ```json
   "Action": [
     "dynamodb:PutItem"
   ]
   ```
   Thành:
   ```json
   "Action": [
     "dynamodb:GetItem"
   ]
   ```
   *(Hành động này cố tình tước quyền ghi `PutItem` của Lambda vào bảng `MediaMetadata`).*
5. Nhấn **Next** $\rightarrow$ Nhấn **Save changes**.
![Test8](/images/5-Workshop/5.10-Testing/Test8.png)
#### Bước 2: Kích hoạt sự cố từ giao diện Web
1. Quay lại trang Web Portal `http://fcaj-media-portal-web-2026.s3-website-ap-southeast-2.amazonaws.com`.
2. Tại phần 1 (Upload), chọn một file ảnh mới (ví dụ: `error-test.png`) $\rightarrow$ Bấm **Tải lên S3 Cloud**.
3. File vẫn được tải lên S3 Data Bucket thành công và phát sinh sự kiện gọi Lambda.

#### Bước 3: Quan sát vết lỗi trên Amazon CloudWatch Logs
1. Mở dịch vụ **CloudWatch** $\rightarrow$ Chọn **Logs** $\rightarrow$ **Log groups**.
2. Nhấp vào `/aws/lambda/process-media-metadata` $\rightarrow$ Mở Log stream mới nhất vừa sinh ra.
3. Quan sát thông báo lỗi đỏ được ghi lại chi tiết:
   ```text
   [ERROR] ClientError: An error occurred (AccessDeniedException) when calling the PutItem operation: User: arn:aws:sts::508266023015:assumed-role/LambdaMediaProcessingRole/process-media-metadata is not authorized to perform: dynamodb:PutItem on resource: arn:aws:dynamodb:ap-southeast-2:508266023015:table/MediaMetadata
   Traceback (most recent call last):
     File "/var/task/lambda_function.py", line 125, in lambda_handler
       table.put_item(Item=...)
   ```

#### Bước 4: Kiểm tra trạng thái CloudWatch Alarm
1. Ở menu bên trái của CloudWatch, chọn **Alarms** $\rightarrow$ **All alarms**.
2. Tìm kiếm alarm **`MediaLambdaFailureAlarm`**.
3. Sau khoảng 1 đến 2 phút, quan sát cột **State**:
   - Trạng thái chuyển từ **OK** sang biểu tượng chấm đỏ **In alarm**.
   - Biểu đồ metric `Errors` vọt lên giá trị `1.0`.

#### Bước 5: Kiểm tra Email cảnh báo sự cố khẩn cấp
1. Mở hòm thư email cá nhân.
2. Nhận được một bức thư khẩn cấp từ AWS Notifications với tiêu đề:
   ```text
   ALARM: "MediaLambdaFailureAlarm" in Asia Pacific (Sydney)
   ```
3. Mở email kiểm tra nội dung: AWS thông báo rõ ràng rằng metric `Errors` của hàm `process-media-metadata` đã vượt ngưỡng $\ge 1$ trong khoảng thời gian đánh giá 300 giây.

#### Bước 6: Khôi phục lại trạng thái bình thường cho hệ thống
1. Quay lại IAM Console $\rightarrow$ Role `LambdaMediaProcessingRole` $\rightarrow$ Edit policy `LambdaMediaPipelinePolicy`.
2. Sửa lại quyền `"dynamodb:GetItem"` thành `"dynamodb:PutItem"`.
3. Nhấn **Save changes**.
4. Sau khoảng 5 phút đánh giá, trạng thái của CloudWatch Alarm `MediaLambdaFailureAlarm` sẽ tự động chuyển từ **In alarm** trở về trạng thái màu xanh lá **OK**.

**Checkpoint:** Chuông báo động CloudWatch Alarm kích hoạt chính xác, email cảnh báo sự cố được gửi tự động, xác thực thành công 100% kịch bản kiểm thử bơm lỗi.

---

## 3. Kết quả mong đợi

- Hoàn thành kiểm thử chức năng end-to-end: Web Portal $\rightarrow$ API Gateway $\rightarrow$ S3 Data $\rightarrow$ Lambda $\rightarrow$ DynamoDB $\rightarrow$ SNS Email.
- Xác thực hoạt động của chữ ký điện tử SigV4 cho cả hai phương thức tải lên trực tiếp (PUT) và tải xuống an toàn (GET).
- Kiểm chứng thành công kịch bản Bơm lỗi: CloudWatch Logs ghi nhận chính xác vết ngoại lệ `AccessDeniedException`, CloudWatch Alarm nhảy trạng thái sang `In alarm` và gửi email khẩn cấp tới quản trị viên.
- Toàn bộ tính năng và cơ chế phục hồi lỗi của hệ thống đã được kiểm chứng đầy đủ, sẵn sàng chuyển sang bước dọn dẹp tài nguyên ở Mục 5.11.