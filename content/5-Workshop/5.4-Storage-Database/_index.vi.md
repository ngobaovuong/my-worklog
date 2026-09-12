---
title : "Thiết lập Lưu trữ & Cơ sở dữ liệu"
date : 2026-09-25
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

### Mục tiêu

Khởi tạo kho lưu trữ đối tượng nhị phân **Amazon S3** bảo mật tuyệt đối với chính sách chặn truy cập công khai (Block Public Access), thiết lập cấu hình **CORS** cho phép máy khách upload trực tiếp từ trình duyệt web, và xây dựng bảng NoSQL **Amazon DynamoDB** để lưu trữ toàn bộ metadata kiểm toán của các tệp tin media.

---

## 1. Cơ sở lý thuyết và Kiến trúc dữ liệu

Tầng dữ liệu của hệ thống được xây dựng theo mô hình phân tách hoàn toàn giữa lưu trữ tệp thô (Binary Object) và lưu trữ dữ liệu có cấu trúc (Metadata):

- **Amazon S3 Data Bucket (Lưu trữ nhị phân riêng tư):**
  - Chuyên dụng chứa ảnh và video gốc do người dùng tải lên.
  - Tuân thủ nghiêm ngặt nguyên tắc bảo mật: **Bật 100% tính năng Block All Public Access**. Không một tệp tin nào có thể bị truy cập công khai từ bên ngoài trừ khi có Presigned URL hợp lệ.
- **Cơ chế Cross-Origin Resource Sharing (CORS) trên S3:**
  - Khi ứng dụng web phía client (chạy từ tên miền khác hoặc chạy từ S3 Web Hosting) gửi một request HTTP `PUT` mang dữ liệu tệp tin trực tiếp lên domain của S3 (`https://fcaj-media-source-demo-2026.s3.ap-southeast-2.amazonaws.com`), trình duyệt web sẽ thực hiện kiểm tra chính sách CORS.
  - Nếu bucket S3 chưa được cấu hình CORS cho phép phương thức `PUT` và header `*`, trình duyệt sẽ ngắt kết nối ngay lập tức và ném ra lỗi phổ biến: `Failed to fetch`.
- **Amazon DynamoDB (Cơ sở dữ liệu NoSQL tốc độ cao):**
  - Quản lý metadata của tệp tin với độ trễ phản hồi dưới 10 mili-giây.
  - **Khóa chính kết hợp (Composite Primary Key):**
    - **Partition Key (`FileId` - String):** Định danh tên hoặc đường dẫn của tệp tin.
    - **Sort Key (`UploadTime` - String):** Mốc thời gian tải lên chuẩn ISO 8601 (UTC), giúp quản lý lịch sử kiểm toán của cùng một tệp nếu được tải lên nhiều lần.

---

## 2. Các bước thực hiện

### Bước 1: Khởi tạo Amazon S3 Data Bucket

1. Đăng nhập vào **AWS Management Console** và xác nhận Region đang chọn là **Asia Pacific (Sydney) ap-southeast-2**.
2. Trên thanh tìm kiếm, gõ `S3` và chọn dịch vụ **S3**.
3. Tại giao diện chính của S3, nhấn nút màu cam **Create bucket**.
![S3](/images/5-Workshop/5.4-Storage-Database/S31.png)
4. Tại mục **General configuration**:
   - **Bucket type**: Chọn **General purpose**.
   - **Bucket name**: Nhập chính xác tên bucket:
     ```text
     fcaj-media-source-demo-2026
     ```
     *(Lưu ý: Tên S3 bucket là duy nhất trên toàn cầu. Nếu tên này đã bị chiếm dụng, bạn có thể thêm hậu tố như `fcaj-media-source-demo-2026-yourname` và lưu ý đồng bộ tên mới này vào code Lambda sau này).*
   - **AWS Region**: Chọn đúng **Asia Pacific (Sydney) ap-southeast-2**.
5. Tại mục **Object Ownership**: Giữ mặc định **ACLs disabled (recommended)**.
6. Tại mục **Block Public Access settings for this bucket**:
   - Giữ nguyên tích chọn **Block all public access** (tất cả 4 ô vuông bên dưới đều được đánh dấu chọn).
7. Các mục **Bucket Versioning**, **Tags**, và **Default encryption**: Giữ nguyên thiết lập mặc định (Amazon S3 managed keys - SSE-S3).
8. Cuộn xuống cuối trang và nhấn nút **Create bucket**.
![S32](/images/5-Workshop/5.4-Storage-Database/S32.png)
![S33](/images/5-Workshop/5.4-Storage-Database/S33.png)
![S34](/images/5-Workshop/5.4-Storage-Database/S34.png)
![S35](/images/5-Workshop/5.4-Storage-Database/S35.png)
**Checkpoint:** Bảng danh sách Buckets xuất hiện `fcaj-media-source-demo-2026` với trạng thái **Objects can be public: Bucket and objects not public** và AWS Region là **ap-southeast-2**.

---

### Bước 2: Cấu hình CORS cho S3 Data Bucket

1. Trong danh sách S3 buckets, nhấp chuột vào tên bucket **`fcaj-media-source-demo-2026`**.
2. Chuyển sang tab **Permissions**.
3. Cuộn xuống phần **Cross-origin resource sharing (CORS)** ở gần cuối trang $\rightarrow$ Nhấn nút **Edit**.
4. Trong khung soạn thảo JSON, dán toàn bộ cấu hình quy tắc CORS sau:

![S36](/images/5-Workshop/5.4-Storage-Database/S36.png)
```json
[
  {
    "AllowedHeaders": [
      "*"
    ],
    "AllowedMethods": [
      "GET",
      "PUT",
      "POST",
      "HEAD"
    ],
    "AllowedOrigins": [
      "*"
    ],
    "ExposeHeaders": [
      "ETag"
    ],
    "MaxAgeSeconds": 3000
  }
]
```

5. Nhấn nút **Save changes**.

**Checkpoint:** Tại mục Cross-origin resource sharing (CORS), cấu hình JSON hiển thị đầy đủ các phương thức `GET, PUT, POST, HEAD` với `AllowedOrigins: *`.

---

### Bước 3: Tạo bảng Amazon DynamoDB

1. Trên thanh tìm kiếm console, gõ `DynamoDB` và chọn dịch vụ **DynamoDB**.
2. Ở thanh điều hướng bên trái, chọn mục **Tables** $\rightarrow$ Nhấn nút màu cam **Create table**.
![DynamoDB1](/images/5-Workshop/5.4-Storage-Database/DynamoDB1.png)
3. Tại trang cấu hình **Create DynamoDB table**:
   - **Table name**: Nhập chính xác:
     ```text
     MediaMetadata
     ```
   - **Partition key**: Nhập `FileId` $\rightarrow$ Kiểu dữ liệu chọn **String**.
   - **Sort key**: Tích chọn ô *Add sort key* $\rightarrow$ Nhập `UploadTime` $\rightarrow$ Kiểu dữ liệu chọn **String**.
4. Tại mục **Table class**: Chọn **DynamoDB Standard**.
5. Tại mục **Capacity specs**: Chọn **Default settings** (Tự động kích hoạt mức Provisioned 5 RCU / 5 WCU hoàn toàn miễn phí theo hạn mức AWS Free Tier).
6. Cuộn xuống cuối trang và nhấn nút **Create table**.
![DynamoDB2](/images/5-Workshop/5.4-Storage-Database/DynamoDB2.png)

```text
Table name: MediaMetadata
Partition key: FileId (String)
Sort key: UploadTime (String)
Table class: DynamoDB Standard
```

7. Chờ khoảng 10 đến 15 giây để bảng hoàn tất quá trình khởi tạo.
![DynamoDB3](/images/5-Workshop/5.4-Storage-Database/DynamoDB3.png)

**Checkpoint:** Trạng thái bảng `MediaMetadata` chuyển sang màu xanh lá **Active**. Nhấp vào tên bảng để kiểm tra chi tiết ARN có dạng: `arn:aws:dynamodb:ap-southeast-2:<account-id>:table/MediaMetadata`.

---

## 3. Kết quả mong đợi

- Khởi tạo thành công S3 Data Bucket `fcaj-media-source-demo-2026` tại vùng Sydney, bảo mật 100% không cho phép public object thô.
- Cấu hình CORS hoàn chỉnh trên S3, sẵn sàng nhận file tải lên qua HTTP `PUT` từ bất kỳ client browser nào thông qua Presigned URL.
- Bảng NoSQL DynamoDB `MediaMetadata` hoạt động ở trạng thái **Active**, sẵn sàng nhận dữ liệu metadata do AWS Lambda ghi vào.
- Hạ tầng lưu trữ và cơ sở dữ liệu đã sẵn sàng để chuyển sang cấu hình dịch vụ thông báo Amazon SNS ở chương tiếp theo.