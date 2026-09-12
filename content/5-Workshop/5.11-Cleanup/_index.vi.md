---
title : "Dọn dẹp tài nguyên"
date : 2026-09-25
weight : 11
chapter : false
pre : " <b> 5.11. </b> "
---

### Mục tiêu

Thực hiện quy trình thu hồi và xóa bỏ hoàn toàn các tài nguyên đám mây đã khởi tạo trong suốt workshop theo đúng thứ tự logic. Việc dọn dẹp giúp tuân thủ triệt để trụ cột **Cost Optimization** của AWS Well-Architected Framework, bảo vệ tài khoản khỏi các chi phí phát sinh ngoài ý muốn sau khi hoàn thành khóa thực tập hoặc khi hết hạn ưu đãi AWS Free Tier.

---

## 1. Cơ sở lý thuyết và Nguyên tắc Dọn dẹp an toàn

Trong quá trình triển khai hạ tầng trên đám mây, các tài nguyên dù ở trạng thái không sử dụng (Idle) vẫn có thể phát sinh chi phí ngầm nếu không được giải phóng đúng quy cách:
- **Lưu trữ S3 & DynamoDB:** Tính phí tích lũy theo dung lượng tệp tin và dung lượng bảng NoSQL được giữ lại theo thời gian.
- **Quy tắc phụ thuộc tài nguyên (Resource Dependencies):** 
  - Amazon S3 không cho phép xóa một Bucket nếu bên trong vẫn còn chứa đối tượng (Objects). Bắt buộc phải thực hiện thao tác **Empty Bucket** (Làm rỗng) trước khi xóa Bucket.
  - Các cấu hình Trigger trên S3 sẽ tự động được thu hồi khi hàm Lambda hoặc Bucket bị gỡ bỏ.
- **Thứ tự dọn dẹp khuyến nghị:** 
  1. Xóa các tầng biên và giao diện người dùng trước (S3 Web Hosting, API Gateway).
  2. Xóa tầng tính toán (AWS Lambda).
  3. Xóa tầng lưu trữ dữ liệu (S3 Data Bucket, DynamoDB Table).
  4. Xóa các dịch vụ giám sát và thông báo (CloudWatch Alarms, CloudWatch Logs, SNS Topic).
  5. Xóa quyền hạn định danh cuối cùng (IAM Role).

---

## 2. Các bước thực hiện chi tiết

### Bước 1: Dọn dẹp và Xóa các Amazon S3 Buckets

Hệ thống có 2 bucket cần xóa: Bucket chứa dữ liệu (`fcaj-media-source-demo-2026`) và Bucket chứa website (`fcaj-media-portal-web-2026`).

#### 1. Xóa S3 Data Bucket (`fcaj-media-source-demo-2026`):
1. Đăng nhập vào **AWS Management Console** $\rightarrow$ Chọn Region **Asia Pacific (Sydney) ap-southeast-2**.
2. Tìm kiếm và mở dịch vụ **Amazon S3**.
3. Trong danh sách Buckets, tích chọn vào ô vuông bên cạnh tên bucket **`fcaj-media-source-demo-2026`**.
4. Nhấn nút **Empty** ở thanh công cụ phía trên.
5. Tại trang xác nhận, nhập cụm từ:
   ```text
   permanently delete
   ```
6. Nhấn nút **Empty** ở góc dưới bên phải để xóa sạch tất cả ảnh/video bên trong bucket.
7. Sau khi làm rỗng thành công, nhấn nút **Exit**.
8. Tích chọn lại vào bucket **`fcaj-media-source-demo-2026`** $\rightarrow$ Nhấn nút **Delete**.
9. Tại ô xác nhận xóa, nhập chính xác tên bucket:
   ```text
   fcaj-media-source-demo-2026
   ```
10. Nhấn nút **Delete bucket**.

#### 2. Xóa S3 Web Hosting Bucket (`fcaj-media-portal-web-2026`):
1. Tương tự, tích chọn vào ô vuông cạnh tên bucket **`fcaj-media-portal-web-2026`**.
2. Nhấn nút **Empty** $\rightarrow$ Nhập `permanently delete` $\rightarrow$ Nhấn **Empty**.
3. Nhấn **Exit** để quay lại danh sách.
4. Tích chọn lại vào bucket **`fcaj-media-portal-web-2026`** $\rightarrow$ Nhấn **Delete**.
5. Nhập tên bucket để xác nhận:
   ```text
   fcaj-media-portal-web-2026
   ```
6. Nhấn nút **Delete bucket**.

**Checkpoint:** Cả 2 buckets biến mất hoàn toàn khỏi danh sách Buckets trên Amazon S3.

---

### Bước 2: Xóa Amazon API Gateway REST API

1. Trên thanh tìm kiếm console, gõ `API Gateway` và chọn dịch vụ **API Gateway**.
2. Trong danh sách APIs, tìm API có tên: **`MediaPortalAPI`**.
3. Tích chọn vào ô vuông trước tên API (hoặc nhấn vào biểu tượng dấu 3 chấm / menu **Actions** cạnh tên API).
4. Chọn **Delete**.
5. Trong hộp thoại xác nhận **Delete API**, nhấn nút **Delete**.

**Checkpoint:** REST API `MediaPortalAPI` bị xóa hoàn toàn, đường dẫn Invoke URL chính thức ngừng hoạt động.

---

### Bước 3: Xóa AWS Lambda Function

1. Trên thanh tìm kiếm, gõ `Lambda` và chọn dịch vụ **Lambda**.
2. Ở thanh điều hướng bên trái, chọn mục **Functions**.
3. Tích chọn vào ô vuông trước hàm: **`process-media-metadata`**.
4. Nhấn nút **Actions** ở góc trên bên phải $\rightarrow$ Chọn **Delete**.
5. Trong hộp thoại xác nhận, nhập chữ:
   ```text
   delete
   ```
6. Nhấn nút **Delete**.

**Checkpoint:** Hàm `process-media-metadata` biến mất khỏi danh sách Functions.

---

### Bước 4: Xóa bảng Amazon DynamoDB

1. Trên thanh tìm kiếm, gõ `DynamoDB` và chọn dịch vụ **DynamoDB**.
2. Ở menu bên trái, chọn mục **Tables**.
3. Tích chọn vào ô vuông trước bảng: **`MediaMetadata`**.
4. Nhấn nút **Delete** ở thanh công cụ phía trên.
5. Trong cửa sổ pop-up xác nhận xóa bảng:
   - Nhập chữ:
     ```text
     confirm
     ```
   - Bỏ tích chọn mục tạo CloudWatch alarm tự động (nếu có).
6. Nhấn nút **Delete table**.

**Checkpoint:** Bảng `MediaMetadata` chuyển sang trạng thái *Deleting* và biến mất hoàn toàn sau vài giây.

---

### Bước 5: Dọn dẹp Amazon CloudWatch (Alarm & Log Group)

#### 1. Xóa CloudWatch Alarm:
1. Mở dịch vụ **Amazon CloudWatch**.
2. Ở menu bên trái, chọn **Alarms** $\rightarrow$ **All alarms**.
3. Tích chọn vào ô vuông trước alarm: **`MediaLambdaFailureAlarm`**.
4. Nhấn nút **Actions** $\rightarrow$ Chọn **Delete**.
5. Nhấn **Delete** một lần nữa để xác nhận.

#### 2. Xóa CloudWatch Log Group:
1. Vẫn trong giao diện CloudWatch, ở menu bên trái chọn **Logs** $\rightarrow$ **Log groups**.
2. Trong ô tìm kiếm, nhập:
   ```text
   /aws/lambda/process-media-metadata
   ```
3. Tích chọn vào ô vuông trước tên log group `/aws/lambda/process-media-metadata`.
4. Nhấn nút **Actions** $\rightarrow$ Chọn **Delete log group(s)**.
5. Nhấn **Delete** trong hộp thoại xác nhận.

**Checkpoint:** Alarm và Log Group của Lambda được xóa sạch sẽ, giải phóng dung lượng lưu trữ CloudWatch Logs.

---

### Bước 6: Xóa Amazon SNS (Topic & Subscriptions)

1. Mở dịch vụ **Amazon SNS**.
2. Ở menu bên trái, chọn **Topics**.
3. Tích chọn vào topic: **`MediaProcessingAlerts`**.
4. Nhấn nút **Delete** ở góc trên bên phải.
5. Nhập cụm từ xác nhận:
   ```text
   delete me
   ```
6. Nhấn nút **Delete**.
7. Tiếp tục chọn mục **Subscriptions** ở menu bên trái:
   - Nếu Subscription gắn với email cá nhân của bạn vẫn còn tồn tại, tích chọn vào nó $\rightarrow$ Nhấn **Delete** $\rightarrow$ Xác nhận xóa.

**Checkpoint:** Topic và Subscription bị xóa hoàn toàn khỏi Amazon SNS.

---

### Bước 7: Xóa AWS IAM Role

1. Mở dịch vụ **IAM** trên AWS Console.
2. Ở menu điều hướng bên trái, chọn mục **Roles**.
3. Trong ô tìm kiếm, nhập: `LambdaMediaProcessingRole`.
4. Tích chọn vào ô vuông trước role **`LambdaMediaProcessingRole`**.
5. Nhấn nút **Delete** ở góc trên bên phải.
6. Trong hộp thoại xác nhận, nhập chính xác tên role:
   ```text
   LambdaMediaProcessingRole
   ```
7. Nhấn nút **Delete**.

**Checkpoint:** Role `LambdaMediaProcessingRole` cùng toàn bộ các Inline Policy đi kèm được xóa hoàn toàn khỏi tài khoản AWS.

---

## 3. Kết quả mong đợi

- Toàn bộ 7 dịch vụ AWS cấu thành hệ thống AWS Media Vault (S3, API Gateway, Lambda, DynamoDB, CloudWatch, SNS, IAM) đã được giải phóng hoàn toàn tại Region Sydney `ap-southeast-2`.
- Không còn bất kỳ tiến trình tính toán ngầm, dung lượng lưu trữ nhị phân hay bảng NoSQL nào tồn tại trên tài khoản.
- Tài khoản AWS trở về trạng thái sạch sẽ tuyệt đối, đảm bảo chi phí vận hành sau workshop là **0.00 USD**.