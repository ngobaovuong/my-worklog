---
title : "Phát triển logic với AWS Lambda"
date : 2026-09-25
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

### Mục tiêu

Xây dựng, cấu hình và triển khai mã nguồn hàm **AWS Lambda** (Python 3.12) đóng vai trò trung tâm tính toán (Compute Core) của hệ thống. Hàm xử lý linh hoạt hai luồng tác vụ: cấp phát **S3 Presigned URL** (PUT/GET) có chữ ký số SigV4 cho máy khách thông qua API Gateway và tự động trích xuất metadata tệp tin, lưu vết vào DynamoDB, đồng thời phát thông báo chi tiết qua Amazon SNS khi nhận sự kiện `s3:ObjectCreated:*`.

---

## 1. Cơ sở lý thuyết và Kiến trúc xử lý

Hàm Lambda được lập trình bằng Python 3.12 và thư viện **Boto3 SDK**, tích hợp kiến trúc điều phối sự kiện kép (Dual-Routing Logic):

### 1. Luồng đồng bộ từ Amazon API Gateway (Synchronous Request-Response):
- Khi người dùng tương tác với giao diện Web Portal (yêu cầu tải lên hoặc tải xuống), API Gateway sẽ chuyển tiếp HTTP Request dưới dạng JSON Proxy Event vào Lambda.
- **Xử lý CORS Preflight (`OPTIONS`):** Trả về ngay lập tức HTTP status 200 kèm các header `Access-Control-Allow-Origin: *` để trình duyệt cho phép giao tiếp.
- **Cấp Presigned URL cho Upload (`PUT`):** Gọi hàm `s3_client.generate_presigned_url(ClientMethod='put_object', ...)` với thời hạn hiệu lực là 300 giây (5 phút). Trình duyệt dùng URL này để tải file trực tiếp lên S3 mà không cần IAM credential.
- **Cấp Presigned URL cho Download (`GET`):** Gọi hàm `generate_presigned_url(ClientMethod='get_object', ...)` tạo liên kết tải về an toàn, hết hạn sau 300 giây.
- **Bài học cấu hình Endpoint SigV4:** Để tránh hiện tượng trình duyệt bị chuyển hướng HTTP 307 và chặn CORS do S3 Global Endpoint, Boto3 client bắt buộc phải cấu hình tường minh:
  ```python
  config = Config(signature_version='s3v4')
  endpoint_url = '[https://s3.ap-southeast-2.amazonaws.com](https://s3.ap-southeast-2.amazonaws.com)'
  ```

### 2. Luồng bất đồng bộ từ Amazon S3 (Asynchronous Event Consumer):
- Khi file được upload thành công vào S3 Data Bucket, S3 tự động kích hoạt Lambda với payload chứa mảng `Records`.
- **Giải mã tên tệp (`urllib.parse.unquote_plus`):** S3 tự động mã hóa URL tên tệp (ví dụ dấu cách đổi thành `+` hoặc `%20`). Việc giải mã giúp lấy chính xác tên file gốc.
- **Ghi dữ liệu kiểm toán vào DynamoDB:** Ghi bản ghi vào bảng `MediaMetadata` gồm `FileId`, `UploadTime`, `Bucket`, `SizeBytes`, `Format`, `Status`.
- **Phát tán thông báo qua SNS:** Định dạng nội dung thông báo thành bảng ASCII trực quan, chứa kích thước tệp tính theo Byte và KB, sau đó publish vào SNS Topic `MediaProcessingAlerts`.

---

## 2. Các bước thực hiện

### Bước 1: Khởi tạo AWS Lambda Function

1. Đăng nhập vào **AWS Management Console** và đảm bảo Region đang chọn là **Asia Pacific (Sydney) ap-southeast-2**.
2. Trên thanh tìm kiếm, gõ `Lambda` và chọn dịch vụ **Lambda**.
3. Tại trang danh sách hàm, nhấn nút màu cam **Create function**.
![Lambda](/images/5-Workshop/5.6-Lambda-Compute/Lambda.png)
4. Chọn tùy chọn **Author from scratch**:
   - **Function name**: Nhập chính xác:
     ```text
     process-media-metadata
     ```
   - **Runtime**: Chọn **Python 3.12**.
   - **Architecture**: Chọn **x86_64**.
5. Mở rộng phần **Change default execution role**:
   - Chọn **Use an existing role**.
   - Tại ô **Existing role**: Chọn role `LambdaMediaProcessingRole` đã cấu hình ở Mục 5.3.
6. Nhấn nút **Create function** ở góc dưới cùng bên phải.
![Lambda2](/images/5-Workshop/5.6-Lambda-Compute/Lambda2.png)
```text
Function name: process-media-metadata
Runtime: Python 3.12
Architecture: x86_64
Execution role: Use an existing role (LambdaMediaProcessingRole)
```

**Checkpoint:** Màn hình chuyển sang trang chi tiết của hàm `process-media-metadata`. Thanh thông báo màu xanh xác nhận khởi tạo thành công.
![Lambda3](/images/5-Workshop/5.6-Lambda-Compute/Lambda3.png)
---

### Bước 2: Triển khai mã nguồn hàm Lambda

1. Tại trang chi tiết hàm, cuộn xuống phần **Code source**.
2. Nhấp đúp chuột vào file `lambda_function.py` trong cây thư mục bên trái.
3. Xóa toàn bộ nội dung mặc định và dán toàn bộ đoạn mã nguồn hoàn chỉnh dưới đây:
![Lambda4](/images/5-Workshop/5.6-Lambda-Compute/Lambda4.png)
![Lambda5](/images/5-Workshop/5.6-Lambda-Compute/Lambda5.png)
![Lambda6](/images/5-Workshop/5.6-Lambda-Compute/Lambda6.png)
```python
import json
import urllib.parse
import boto3
from datetime import datetime
from botocore.config import Config

# Cấu hình Boto3 Client với SigV4 và ép Endpoint vùng Sydney
s3_client = boto3.client(
    's3',
    region_name='ap-southeast-2',
    endpoint_url='[https://s3.ap-southeast-2.amazonaws.com](https://s3.ap-southeast-2.amazonaws.com)',
    config=Config(signature_version='s3v4')
)
dynamodb = boto3.resource('dynamodb', region_name='ap-southeast-2')
sns_client = boto3.client('sns', region_name='ap-southeast-2')

# Khai báo các tài nguyên đích
BUCKET_NAME = "fcaj-media-source-demo-2026"
TABLE_NAME = "MediaMetadata"
# THAY THẾ CHUỖI DƯỚI ĐÂY BẰNG ARN TOPIC SNS THỰC TẾ CỦA BẠN
SNS_TOPIC_ARN = "arn:aws:sns:ap-southeast-2:508266023015:MediaProcessingAlerts"

def lambda_handler(event, context):
    headers = {
        "Access-Control-Allow-Origin": "*",
        "Access-Control-Allow-Headers": "Content-Type",
        "Access-Control-Allow-Methods": "OPTIONS,POST,GET"
    }

    # =========================================================================
    # 1. NHÁNH XỬ LÝ REQUEST TỪ API GATEWAY (Sinh Presigned URL)
    # =========================================================================
    if 'httpMethod' in event or 'requestContext' in event:
        http_method = event.get('httpMethod', '')
        
        # Xử lý CORS Preflight handshake
        if http_method == 'OPTIONS':
            return {
                'statusCode': 200,
                'headers': headers,
                'body': ''
            }
            
        body = json.loads(event.get('body', '{}')) if event.get('body') else {}
        action = body.get('action')
        file_name = body.get('fileName')

        if not file_name:
            return {
                'statusCode': 400,
                'headers': headers,
                'body': json.dumps({'error': 'fileName is required in request payload'})
            }

        # Sinh URL tạm thời để Upload (PUT)
        if action == 'get_upload_url':
            presigned_url = s3_client.generate_presigned_url(
                ClientMethod='put_object',
                Params={
                    'Bucket': BUCKET_NAME,
                    'Key': file_name
                },
                ExpiresIn=300
            )
            return {
                'statusCode': 200,
                'headers': headers,
                'body': json.dumps({
                    'uploadUrl': presigned_url,
                    'fileName': file_name
                })
            }

        # Sinh URL tạm thời để Download (GET)
        elif action == 'get_download_url':
            presigned_url = s3_client.generate_presigned_url(
                ClientMethod='get_object',
                Params={
                    'Bucket': BUCKET_NAME,
                    'Key': file_name
                },
                ExpiresIn=300
            )
            return {
                'statusCode': 200,
                'headers': headers,
                'body': json.dumps({
                    'downloadUrl': presigned_url
                })
            }

    # =========================================================================
    # 2. NHÁNH XỬ LÝ SỰ KIỆN TỰ ĐỘNG TỪ S3 (Khi tệp tải lên hoàn tất)
    # =========================================================================
    if 'Records' in event and 's3' in event['Records'][0]:
        table = dynamodb.Table(TABLE_NAME)
        
        for record in event['Records']:
            bucket = record['s3']['bucket']['name']
            # Giải mã URL an toàn (xử lý khoảng trắng và ký tự đặc biệt)
            key = urllib.parse.unquote_plus(record['s3']['object']['key'])
            size_bytes = record['s3']['object']['size']
            upload_time = datetime.utcnow().strftime('%Y-%m-%d %H:%M:%S UTC')
            file_extension = key.split('.')[-1].upper() if '.' in key else 'UNKNOWN'

            # Ghi thông tin kiểm toán vào DynamoDB
            table.put_item(
                Item={
                    'FileId': key,
                    'UploadTime': upload_time,
                    'Bucket': bucket,
                    'SizeBytes': size_bytes,
                    'Format': file_extension,
                    'Status': 'ACTIVE'
                }
            )

            # Định dạng nội dung thông báo gửi qua Amazon SNS
            size_kb = round(size_bytes / 1024, 2)
            detailed_msg = (
                "===========================================\n"
                "       AWS MEDIA VAULT - NEW UPLOAD        \n"
                "===========================================\n"
                f"• Tên tệp tin    : {key}\n"
                f"• Định dạng      : {file_extension}\n"
                f"• Dung lượng     : {size_bytes} Bytes (~{size_kb} KB)\n"
                f"• Kho lưu trữ    : {bucket}\n"
                f"• Thời gian tải  : {upload_time}\n"
                f"• Cơ sở dữ liệu  : DynamoDB [MediaMetadata] - SUCCESS\n"
                "===========================================\n"
                "Trạng thái: Tệp đã lưu trữ thành công và sẵn sàng truy xuất an toàn."
            )

            sns_client.publish(
                TopicArn=SNS_TOPIC_ARN,
                Subject=f"[AWS Media Vault] File {key} Processed Successfully",
                Message=detailed_msg
            )

        return {
            'statusCode': 200,
            'body': json.dumps('S3 Event Handled and Processed Successfully')
        }

    return {
        'statusCode': 400,
        'headers': headers,
        'body': json.dumps({'error': 'Invalid event trigger or unhandled route'})
    }
```

4. Kiểm tra lại chuỗi `SNS_TOPIC_ARN` và `BUCKET_NAME` đảm bảo khớp 100% với tài nguyên của bạn.
5. Nhấn nút màu xanh dương **Deploy** ở thanh công cụ phía trên khung soạn thảo mã nguồn.

**Checkpoint:** Dòng thông báo màu xanh `Successfully deployed changes` xuất hiện.

---

### Bước 3: Cấu hình Trigger tự động từ S3 Bucket

1. Tại khung **Function overview** ở đầu trang cấu hình Lambda, nhấn nút **+ Add trigger**.
2. Tại ô **Select a source**: Nhập `S3` và chọn **S3**.
3. Cấu hình các thông số Trigger:
   - **Bucket**: Chọn `fcaj-media-source-demo-2026`.
   - **Event types**: Chọn **All object create events** (`s3:ObjectCreated:*`).
   - **Prefix**: Bỏ trống.
   - **Suffix**: Bỏ trống (cho phép bắt tất cả định dạng ảnh và video).
4. Tích chọn vào ô vuông cảnh báo đệ quy:
   - *"I understand that using the same S3 bucket for input and output is not recommended and can cause recursive invocations."*
5. Nhấn nút **Add**.

```text
Trigger Source: S3
Bucket: fcaj-media-source-demo-2026
Event type: All object create events
Recursive invocation acknowledgment: Checked
```

**Checkpoint:** Trong phần sơ đồ **Function overview**, icon **S3** xuất hiện ở nhánh bên trái liên kết trực tiếp vào hàm Lambda.

---

### Bước 4: Kiểm thử đơn vị trực tiếp (Unit Testing) trên Lambda Console

Để chứng minh hàm hoạt động tốt độc lập và kiểm tra quyền IAM trước khi dựng API Gateway:

1. Chuyển sang tab **Test** (nằm cạnh tab Code).
2. Tại mục **Test event action**: Chọn **Create new event**.
3. **Event name**: Nhập `TestS3UploadEvent`.
4. Tại ô **Template**: Gõ tìm kiếm và chọn mẫu **s3-put**.
5. Trong khung JSON mẫu của sự kiện, sửa 2 trường sau:
   - Tìm `"name": "example-bucket"` $\rightarrow$ Đổi thành: `"name": "fcaj-media-source-demo-2026"`
   - Tìm `"key": "test%2Fkey"` $\rightarrow$ Đổi thành tên file bất kỳ: `"key": "test-manual-upload.png"`
6. Nhấn nút **Save** ở góc trên, sau đó nhấn nút **Test** màu cam.
7. Quan sát kết quả hiển thị:
   - Khung kết quả trả về màu xanh lá: **Execution result: Succeeded (Status: 200)**.
   - Kiểm tra bảng DynamoDB `MediaMetadata`: Bản ghi `test-manual-upload.png` đã xuất hiện.
   - Kiểm tra Email cá nhân: Nhận được email báo cáo tự động từ SNS.

**Checkpoint:** Unit test thành công với Status 200, xác nhận hàm Lambda đã có đủ quyền và sẵn sàng kết nối vào API Gateway.

---

## 3. Kết quả mong đợi

- Khởi tạo thành công hàm Lambda `process-media-metadata` sử dụng runtime Python 3.12 tại Region Sydney `ap-southeast-2`.
- Mã nguồn triển khai hỗ trợ cả 2 chế độ: sinh chữ ký số S3 Presigned URL và tiêu thụ sự kiện S3 tự động.
- S3 Bucket `fcaj-media-source-demo-2026` được liên kết trực tiếp làm Trigger sự kiện vào Lambda.
- Đã kiểm thử thành công qua Test Event trên console, xác nhận không còn lỗi phân quyền Permissions Boundary và sẵn sàng chuyển sang tích hợp cổng kết nối Amazon API Gateway ở Mục 5.7.