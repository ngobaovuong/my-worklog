---
title : "Xây dựng cổng kết nối API Gateway"
date : 2026-09-13
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

### Mục tiêu

Khởi tạo, cấu hình và triển khai dịch vụ **Amazon API Gateway** (REST API) làm tầng giao tiếp bảo mật giữa máy khách (Web Portal) và hàm AWS Lambda. Xử lý triệt để bài toán **CORS** (Cross-Origin Resource Sharing) đa tầng và thiết lập cơ chế **Lambda Proxy Integration** để định tuyến yêu cầu tạo S3 Presigned URL.

---

## 1. Cơ sở lý thuyết và Kiến trúc kết nối

Trong kiến trúc Serverless hiện đại, máy khách không bao giờ gọi trực tiếp vào hàm Lambda thông qua AWS SDK nội bộ (vì sẽ làm lộ IAM Credentials trên trình duyệt). Thay vào đó, **Amazon API Gateway** đóng vai trò là reverse proxy bảo mật:

- **Mô hình REST API Regional:** API endpoint được triển khai cùng Region với Lambda và S3 (**Sydney `ap-southeast-2`**) nhằm tối ưu hóa độ trễ truyền gói tin mạng.
- **Cơ chế Lambda Proxy Integration:**
  - Toàn bộ HTTP request (gồm Headers, Query Parameters, HTTP Method, Request Body) được đóng gói nguyên vẹn thành một JSON payload và chuyển giao trực tiếp cho hàm Lambda thông qua tham số `event`.
  - Giúp loại bỏ sự phụ thuộc vào các bộ chuyển đổi dữ liệu (Mapping Templates) phức tạp, giúp backend Python kiểm soát hoàn toàn logic điều hướng.
- **Bản chất của lỗi `Failed to fetch` và giải pháp CORS Preflight:**
  - Khi trình duyệt web gửi một request POST/PUT mang header tùy biến (`Content-Type: application/json`) từ một domain khác domain của API Gateway (ví dụ từ S3 Website Hosting hoặc mở file `index.html` cục bộ), trình duyệt sẽ tự động gửi một gói tin thăm dò gọi là **Preflight Request** sử dụng HTTP method `OPTIONS`.
  - Nếu API Gateway không có method `OPTIONS` hoặc không trả về các header sau:
    ```text
    Access-Control-Allow-Origin: *
    Access-Control-Allow-Headers: Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token
    Access-Control-Allow-Methods: OPTIONS,POST,GET
    ```
    Trình duyệt sẽ ngay lập tức chặn kết nối và thông báo lỗi `Failed to fetch` trên console mà không cho phép dữ liệu đi tới Lambda.
  - *Lưu ý quan trọng:* Mỗi khi thay đổi cấu hình Resource, Method hoặc CORS trên API Gateway, bạn **bắt buộc phải Deploy API** lại vào Stage (`prod`) thì thay đổi mới có hiệu lực thực tế trên Internet.

---

## 2. Các bước thực hiện

### Bước 1: Khởi tạo REST API trên Amazon API Gateway

1. Đăng nhập vào **AWS Management Console** (xác nhận Region góc trên bên phải là **Asia Pacific (Sydney) ap-southeast-2**).
2. Trên thanh tìm kiếm, gõ `API Gateway` và chọn dịch vụ **API Gateway**.
![APIGateway](/images/5-Workshop/5.7-API-Gateway/APIGateway.png)
3. Tại trang tổng quan dịch vụ, tìm khối **REST API** (chú ý: chọn ô *REST API* thông thường, **không chọn** *REST API (Private)*) $\rightarrow$ Nhấn nút **Build**.
4. Tại trang cấu hình **Create REST API**:
   - **Choose the protocol**: Chọn **REST**.
   - **Create new API**: Chọn **New API**.
   - **API name**: Nhập chính xác:
     ```text
     MediaPortalAPI
     ```
   - **Description**: Nhập:
     ```text
     API Gateway for AWS Media Vault to issue S3 Presigned URLs
     ```
   - **Endpoint Type**: Chọn **Regional** (để cùng vùng Sydney với Lambda).
5. Nhấn nút màu cam **Create API**.
![APIGateway2](/images/5-Workshop/5.7-API-Gateway/APIGateway2.png)
```text
Protocol: REST
Create new API: New API
API name: MediaPortalAPI
Endpoint Type: Regional
```

**Checkpoint:** Màn hình chuyển sang giao diện quản lý tài nguyên của `MediaPortalAPI`. Cột Resources hiện chỉ có thư mục gốc `/`.
![APIGateway3](/images/5-Workshop/5.7-API-Gateway/APIGateway3.png)
---

### Bước 2: Tạo Resource `/media`

1. Nhấp chuột vào dấu gạch chéo `/` (root resource) ở cây thư mục bên trái.
2. Nhấn nút **Create resource** ở thanh công cụ phía trên.
3. Tại trang cấu hình **Resource details**:
   - **Resource path**: Mặc định là `/`.
   - **Resource name**: Nhập `media`.
   - **Resource path sau khi nhập**: Sẽ tự động điền thành `/media`.
   - **CORS (Cross-Origin Resource Sharing)**: **Tích chọn** vào ô vuông này (AWS sẽ tự động sinh cấu hình cơ bản).
4. Nhấn nút **Create resource**.
![APIGateway4](/images/5-Workshop/5.7-API-Gateway/APIGateway4.png)
**Checkpoint:** Cây thư mục xuất hiện nhánh con `/media` nằm dưới `/`.

---

### Bước 3: Tạo Method `ANY` với Lambda Proxy Integration

1. Nhấp chuột vào nhánh `/media` vừa tạo.
2. Nhấn nút **Create method** ở thanh công cụ phía trên.
3. Tại trang cấu hình **Method details**:
   - **Method type**: Chọn **ANY** từ danh sách thả xuống (Method `ANY` giúp tiếp nhận mọi phương thức HTTP như GET, POST, OPTIONS chuyển tiếp về Lambda).
   - **Integration type**: Chọn **Lambda function**.
   - **Lambda proxy integration**: **Bật thanh gạt (Toggle On)** sang màu cam *(Đây là bước bắt buộc để Lambda đọc được `event['body']` và `event['httpMethod']`)*.
   - **Lambda Region**: Chọn **ap-southeast-2** (Sydney).
   - **Lambda function**: Gõ tìm và chọn đúng hàm:
     ```text
     process-media-metadata
     ```
4. Nhấn nút màu cam **Create method**.
5. Nếu một hộp thoại xuất hiện yêu cầu cấp quyền cho API Gateway gọi hàm Lambda (`Add Permission to Lambda Function`), nhấn **OK**.

```text
Method type: ANY
Integration type: Lambda function
Lambda proxy integration: Checked (Enabled)
Lambda function: process-media-metadata
```

**Checkpoint:** Trong nhánh `/media`, method `ANY` xuất hiện. Bấm vào `ANY` sẽ thấy sơ đồ luồng luân chuyển dữ liệu: `Client` $\rightarrow$ `Method Request` $\rightarrow$ `Integration Request (Lambda Proxy)` $\rightarrow$ `process-media-metadata`.

---

### Bước 4: Kích hoạt và Chuẩn hóa CORS cho Resource `/media`

Để đảm bảo trình duyệt web không bị chặn khi gọi request POST và OPTIONS:

1. Nhấp chuột chọn vào resource `/media`.
2. Nhấn nút **Enable CORS** trên thanh công cụ phía trên.
3. Cấu hình các giá trị CORS:
   - **Gateway responses**: Tích chọn **Default 4XX** và **Default 5XX** (để các lỗi từ Gateway cũng trả về header CORS).
   - **Methods**: Tích chọn tất cả các method hiển thị (ít nhất phải có **ANY** và **OPTIONS**).
   - **Access-Control-Allow-Origin**: Giữ nguyên `'*'`.
   - **Access-Control-Allow-Headers**: Giữ mặc định `'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'`.
4. Nhấn nút **Save** ở góc dưới cùng bên phải để lưu cấu hình.

**Checkpoint:** Dưới nhánh `/media`, phương thức `OPTIONS` xuất hiện riêng biệt với phản hồi Mock 200 tự động kèm theo các header CORS đã được thiết lập.

---

### Bước 5: Triển khai API lên Stage `prod`

Mọi thay đổi cấu hình API Gateway chỉ có hiệu lực ra ngoài Internet sau khi thực hiện thao tác Deploy API:

1. Tại thanh công cụ phía trên, nhấn nút màu cam **Deploy API**.
2. Một cửa sổ pop-up **Deploy API** xuất hiện:
   - **Stage**: Chọn **\*New Stage\***.
   - **Stage name**: Nhập chính xác:
     ```text
     prod
     ```
   - **Deployment description**: Nhập:
     ```text
     Production deployment for AWS Media Vault
     ```
3. Nhấn nút **Deploy**.
![APIGateway5](/images/5-Workshop/5.7-API-Gateway/APIGateway5.png)
**Checkpoint:** Console tự động chuyển sang tab **Stages** $\rightarrow$ `prod`.

---

### Bước 6: Lấy Invoke URL và Kiểm tra Endpoint

1. Tại cây thư mục của Stage `prod`, mở rộng nhánh `prod` $\rightarrow$ Nhấp chuột vào resource `/media`.
2. Quan sát dòng chữ màu xanh dương **Invoke URL** ở đầu trang. Đường dẫn chuẩn sẽ có định dạng:
   ```text
   https://<api-id>[.execute-api.ap-southeast-2.amazonaws.com/prod/media](https://.execute-api.ap-southeast-2.amazonaws.com/prod/media)
   ```
   *(Ví dụ: `https://a1b2c3d4e5.execute-api.ap-southeast-2.amazonaws.com/prod/media`)*.
3. **Sao chép lại đường dẫn này** vào Notepad. Đây chính là giá trị hằng số `API_ENDPOINT` sẽ được sử dụng trong file giao diện `index.html` ở Mục 5.8.
![APIGateway6](/images/5-Workshop/5.7-API-Gateway/APIGateway6.png)
#### Kiểm tra nhanh bằng Terminal (cURL Test):
Mở terminal trên máy tính và chạy lệnh kiểm tra xem API Gateway đã kết nối thành công tới Lambda hay chưa:

```bash
curl -X POST https://<api-id>[.execute-api.ap-southeast-2.amazonaws.com/prod/media](https://.execute-api.ap-southeast-2.amazonaws.com/prod/media) \
  -H "Content-Type: application/json" \
  -d '{"action": "get_upload_url", "fileName": "ping-test.png"}'
```

**Phản hồi kỳ vọng từ API Gateway:**
```json
{"uploadUrl": "[https://fcaj-media-source-demo-2026.s3.ap-southeast-2.amazonaws.com/ping-test.png?X-Amz-Algorithm=AWS4-HMAC-SHA256](https://fcaj-media-source-demo-2026.s3.ap-southeast-2.amazonaws.com/ping-test.png?X-Amz-Algorithm=AWS4-HMAC-SHA256)...", "fileName": "ping-test.png"}
```

**Checkpoint:** Lệnh cURL trả về JSON chứa `uploadUrl` có chữ ký SigV4 hợp lệ, xác nhận API Gateway và Lambda đã tích hợp thành công 100%.

---

## 3. Kết quả mong đợi

- Khởi tạo thành công REST API `MediaPortalAPI` loại Regional tại Region Sydney `ap-southeast-2`.
- Resource `/media` được liên kết với hàm Lambda `process-media-metadata` thông qua method `ANY` bằng cơ chế Lambda Proxy Integration.
- Thiết lập hoàn chỉnh cơ chế CORS cho phương thức `OPTIONS` và `ANY`, loại bỏ hoàn toàn nguy cơ lỗi `Failed to fetch`.
- API đã được Deploy thành công lên stage `prod` và sở hữu đường dẫn Invoke URL hoạt động bình thường, sẵn sàng đưa vào file giao diện Web Portal ở Mục 5.8.