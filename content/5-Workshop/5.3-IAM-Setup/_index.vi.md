---
title : "Cấu hình phân quyền IAM"
date : 2026-09-25
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

### Mục tiêu

Xây dựng và cấu hình **IAM Execution Role** cho hàm AWS Lambda tuân thủ chặt chẽ nguyên tắc đặc quyền tối thiểu (Least Privilege). Đồng thời nhận diện, phòng tránh và xử lý triệt để lỗi hạn chế quyền do **Permissions Boundary** gây ra, đảm bảo hàm Lambda có thể tương tác an toàn với S3, DynamoDB, SNS và CloudWatch Logs.

---

## 1. Cơ sở lý thuyết và Kiến trúc IAM

Trong mô hình kiến trúc Serverless, AWS Lambda đóng vai trò là động cơ xử lý trung gian. Theo mặc định, một hàm Lambda mới tạo sẽ không có quyền hạn truy cập vào bất kỳ dịch vụ lưu trữ hay cơ sở dữ liệu nào của AWS.

Hệ thống cần phân tách rõ ràng các quyền cần thiết:
- **CloudWatch Logs:** Cần quyền `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents` để ghi nhật ký hoạt động. Ta sử dụng AWS Managed Policy: `AWSLambdaBasicExecutionRole`.
- **Amazon S3:** Cần quyền `s3:GetObject` (để đọc file) và `s3:PutObject` (để sinh Presigned URL upload). Chỉ giới hạn trong bucket dữ liệu `fcaj-media-source-*`.
- **Amazon DynamoDB:** Cần quyền `dynamodb:PutItem` để lưu metadata tệp tin vào bảng `MediaMetadata`.
- **Amazon SNS:** Cần quyền `sns:Publish` để gửi bản tin thông báo tới Topic `MediaProcessingAlerts`.

### Bài học khắc phục lỗi Permissions Boundary:
Trong một số môi trường tài khoản trường học hoặc doanh nghiệp, IAM Role có thể bị gán một **Permissions Boundary** ngầm. Permissions Boundary hoạt động như một bộ lọc trần: dù Identity-based Policy đã cấp quyền `dynamodb:PutItem`, nhưng nếu Boundary không khai báo hành động này thì Lambda vẫn bị chặn với thông báo lỗi:
`...is not authorized to perform: dynamodb:PutItem ... because no permissions boundary allows the dynamodb:PutItem action`.
Vì vậy, bài thực hành này yêu cầu kiểm tra và gỡ bỏ hoàn toàn Permissions Boundary khỏi Role.

---

## 2. Các bước thực hiện

### Bước 1: Khởi tạo IAM Role cho Lambda

1. Truy cập vào **AWS Management Console** (đảm bảo đang ở Region **ap-southeast-2 Sydney**).
2. Trên thanh tìm kiếm ở đầu trang, gõ `IAM` và chọn dịch vụ **IAM**.
![Trang IAM](/images/5-Workshop/5.3-IAM-Setup/IAM1.png)
3. Ở menu điều hướng bên trái, chọn mục **Roles** $\rightarrow$ Nhấn nút màu cam **Create role**.
4. Tại trang **Select trusted entity**:
   - **Trusted entity type**: Chọn **AWS service**.
   - **Use case**: Chọn **Lambda** từ danh sách radio button.
5. Nhấn nút **Next** ở góc dưới bên phải.
![Trang IAM Role](/images/5-Workshop/5.3-IAM-Setup/IAM2.png)
```text
Trusted entity type: AWS service
Use case: Lambda
```

**Checkpoint:** Màn hình chuyển sang trang **Add permissions**.

---

### Bước 2: Gán Managed Policy cơ bản (CloudWatch Logs)

1. Tại ô tìm kiếm chính sách (Search policies), nhập: `AWSLambdaBasicExecutionRole`.
2. Nhấn Enter và tích chọn vào ô vuông trước tên policy **AWSLambdaBasicExecutionRole**.
3. Nhấn nút **Next**.
![Trang Add permissions](/images/5-Workshop/5.3-IAM-Setup/IAM4.png)

**Checkpoint:** Màn hình chuyển sang trang **Name, review, and create**.

---

### Bước 3: Đặt tên Role và Kiểm tra Permissions Boundary

1. Tại ô **Role name**: Nhập chính xác tên sau:
   ```text
   LambdaMediaProcessingRole
   ```
2. Tại ô **Description**: Nhập mô tả mục đích sử dụng:
   ```text
   Role for Lambda to process S3 media events, write to DynamoDB, and publish to SNS.
   ```
3. Cuộn xuống phần **Permissions boundary**:
   - Kiểm tra kỹ dòng hiển thị: Phải là **No boundary set**.
   - *Lưu ý quan trọng:* Nếu xuất hiện một boundary đang được chọn, hãy nhấp vào nút gỡ bỏ hoặc chọn không áp dụng boundary.
4. Cuộn xuống cuối cùng và nhấn nút **Create role**.

**Checkpoint:** Thông báo màu xanh hiển thị `Role LambdaMediaProcessingRole created`. Tìm kiếm `LambdaMediaProcessingRole` trong danh sách Roles và nhấp chuột vào tên role để mở trang cấu hình chi tiết.

---

### Bước 4: Tạo Inline Policy cho S3, DynamoDB và SNS

1. Trong trang chi tiết của role `LambdaMediaProcessingRole`, chuyển đến tab **Permissions**.
2. Nhấp vào nút **Add permissions** $\rightarrow$ Chọn **Create inline policy**.
3. Tại giao diện soạn thảo Policy, nhấp chọn tab **JSON** ở góc trên bên phải khung soạn thảo.
4. Xóa toàn bộ đoạn JSON mặc định và dán chính xác đoạn mã định nghĩa chính sách sau:
![Trang IAM Policy](/images/5-Workshop/5.3-IAM-Setup/IAM3.png)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3ObjectAccess",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::fcaj-media-source-*/*"
    },
    {
      "Sid": "DynamoDBAccess",
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-2:*:table/MediaMetadata"
    },
    {
      "Sid": "SNSPublishAccess",
      "Effect": "Allow",
      "Action": [
        "sns:Publish"
      ],
      "Resource": "arn:aws:sns:ap-southeast-2:*:MediaProcessingAlerts"
    }
  ]
}
```

5. Nhấn nút **Next** ở góc dưới bên phải.
6. Tại trang **Review policy**:
   - **Policy name**: Nhập tên chính sách:
     ```text
     LambdaMediaPipelinePolicy
     ```
7. Nhấn nút **Create policy**.

![Trang2 IAM Policy](/images/5-Workshop/5.3-IAM-Setup/IAM5.png)

**Checkpoint:** Tại tab **Permissions** của role `LambdaMediaProcessingRole`, danh sách Permissions policies hiển thị đầy đủ 2 chính sách:
- `AWSLambdaBasicExecutionRole` (AWS managed policy)
- `LambdaMediaPipelinePolicy` (Customer inline policy)

---

## 3. Kết quả mong đợi

- Tạo thành công IAM Role mang tên `LambdaMediaProcessingRole` với Trust Relationship trỏ về `lambda.amazonaws.com`.
- Role sở hữu đầy đủ quyền hạn để tự động tạo CloudWatch Log Group, đọc/ghi S3 object, ghi bản ghi DynamoDB và phát tin qua SNS Topic.
- Không tồn tại bất kỳ Permissions Boundary nào gây cản trở quyền thực thi của Lambda.
- Hạ tầng bảo mật định danh đã sẵn sàng để chuyển sang chương khởi tạo S3 Bucket và bảng dữ liệu DynamoDB.
![Role](/images/5-Workshop/5.3-IAM-Setup/IAM6.png)