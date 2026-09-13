---
title : "Triển khai Web Hosting trên S3"
date : 2026-09-13
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

### Mục tiêu

Khởi tạo và cấu hình một **S3 Bucket riêng biệt** đóng vai trò làm máy chủ phân phối giao diện web tĩnh thông qua tính năng **S3 Static Website Hosting**. Thiết lập chính sách bảo mật **Bucket Policy (Public Read)**, triển khai toàn bộ mã nguồn giao diện máy khách (`index.html`) hỗ trợ tải lên và tải xuống tệp media trực tiếp qua API Gateway và S3 Presigned URL.

---

## 1. Cơ sở lý thuyết và Kiến trúc phân tách bảo mật

Theo trụ cột **Security** và **Cost Optimization** của AWS Well-Architected Framework:

- **Phân tách vùng bảo mật (Security Tier Decoupling):**
  - Tuyệt đối không lưu file giao diện `index.html` vào cùng bucket chứa dữ liệu ảnh/video (`fcaj-media-source-demo-2026`).
  - Bucket dữ liệu phải giữ trạng thái **Block 100% Public Access** để tránh rò rỉ dữ liệu nhạy cảm.
  - Bucket web (`fcaj-media-portal-web-2026`) được mở công khai chỉ với quyền đọc tệp tĩnh (`s3:GetObject`), đóng vai trò như một Web Server không máy chủ (Serverless Web Server).
- **Cơ chế hoạt động của S3 Static Website Hosting:**
  - S3 cung cấp sẵn một HTTP Web Endpoint toàn cầu dạng: `http://<bucket-name>.s3-website-<region>.amazonaws.com`.
  - Không tiêu tốn tài nguyên nhàn rỗi (Zero Idle Cost), miễn phí kích hoạt tính năng và nằm trọn trong hạn mức AWS Free Tier (5 GB lưu trữ, 20.000 GET requests/tháng).
- **Luồng tích hợp trực tiếp từ trình duyệt:**
  1. Người dùng mở trình duyệt, tải file `index.html` từ S3 Hosting Bucket.
  2. JavaScript trên trình duyệt gọi API Gateway `/media` để xin chữ ký số Presigned URL.
  3. Trình duyệt gửi trực tiếp dữ liệu nhị phân lên S3 Data Bucket qua HTTP `PUT`.

---

## 2. Các bước thực hiện

### Bước 1: Khởi tạo S3 Bucket chứa Website

1. Đăng nhập vào **AWS Management Console** (đảm bảo đang ở Region **ap-southeast-2 Sydney**).
2. Trên thanh tìm kiếm, gõ `S3` và chọn dịch vụ **S3**.
3. Tại trang danh sách Buckets, nhấn nút màu cam **Create bucket**.
4. Cấu hình các thông số tạo bucket:
   - **Bucket type**: Chọn **General purpose**.
   - **Bucket name**: Nhập tên duy nhất trên toàn cầu:
     ```text
     fcaj-media-portal-web-2026
     ```
     *(Lưu ý: Nếu tên bị trùng, bạn có thể thêm hậu tố như `fcaj-media-portal-web-2026-yourname`).*
   - **AWS Region**: Chọn đúng **Asia Pacific (Sydney) ap-southeast-2**.
5. Tại mục **Object Ownership**: Chọn **ACLs disabled (recommended)**.
6. Tại mục **Block Public Access settings for this bucket**:
   - **Bỏ tích chọn** ở ô **Block all public access** (để mở quyền cho phép gán Bucket Policy đọc công khai).
   - Xuất hiện cảnh báo màu vàng, **tích chọn vào ô xác nhận**:
     *"I acknowledge that the current settings might result in this bucket and the objects within becoming public."*
7. Giữ mặc định các thông số còn lại $\rightarrow$ Cuộn xuống cuối trang và nhấn nút **Create bucket**.

```text
Bucket name: fcaj-media-portal-web-2026
AWS Region: ap-southeast-2 (Sydney)
Block all public access: Unchecked
Acknowledge public risk: Checked
```

**Checkpoint:** Bucket `fcaj-media-portal-web-2026` được tạo thành công, cột *Objects can be public* hiển thị trạng thái cảnh báo màu cam: **Objects can be public**.
![Web-Hosting](/images/5-Workshop/5.8-Web-Hosting/S3-web-1.png)
---

### Bước 2: Bật tính năng Static Website Hosting

1. Nhấp chuột vào tên bucket **`fcaj-media-portal-web-2026`** vừa tạo.
2. Chuyển sang tab **Properties**.
3. Cuộn xuống cuối trang tìm mục **Static website hosting** $\rightarrow$ Nhấn nút **Edit**.
4. Cấu hình các thông số:
   - **Static website hosting**: Chọn **Enable**.
   - **Hosting type**: Chọn **Host a static website**.
   - **Index document**: Nhập chính xác tên file:
     ```text
     index.html
     ```
   - **Error document**: Nhập `index.html` (tùy chọn).
![Web-Hosting2](/images/5-Workshop/5.8-Web-Hosting/S3-web-2.png)
5. Nhấn nút **Save changes**.
6. Sau khi lưu, cuộn lại xuống mục **Static website hosting**, quan sát dòng **Bucket website endpoint** có định dạng:
   ```text
   [http://fcaj-media-portal-web-2026.s3-website-ap-southeast-2.amazonaws.com](http://fcaj-media-portal-web-2026.s3-website-ap-southeast-2.amazonaws.com)
   ```
7. Sao chép lại đường link này vào Notepad để truy cập sau khi upload code.

**Checkpoint:** Mục Static website hosting hiển thị trạng thái **Enabled** kèm theo đường dẫn HTTP endpoint hợp lệ.
![Web-Hosting3](/images/5-Workshop/5.8-Web-Hosting/S3-web-3.png)
---

### Bước 3: Cấu hình Bucket Policy công khai (Read-Only)

Mặc dù đã tắt Block Public Access, bucket vẫn cần một chính sách cho phép mọi người dùng Internet được đọc file tĩnh:

1. Chuyển sang tab **Permissions** của bucket `fcaj-media-portal-web-2026`.
2. Cuộn xuống mục **Bucket policy** $\rightarrow$ Nhấn nút **Edit**.
3. Trong khung soạn thảo JSON, dán toàn bộ đoạn policy sau *(Lưu ý: Nếu bạn đặt tên bucket khác, hãy thay đúng tên ở dòng Resource)*:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObjectForWebsite",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::fcaj-media-portal-web-2026/*"
    }
  ]
}
```

4. Nhấn nút **Save changes** ở góc dưới cùng bên phải.

**Checkpoint:** Dưới tên bucket xuất hiện huy hiệu màu đỏ **Public**, xác nhận bucket đã sẵn sàng phân phối tệp tĩnh tới người dùng.
![Web-Hosting4](/images/5-Workshop/5.8-Web-Hosting/S3-web-4.png)
![Web-Hosting5](/images/5-Workshop/5.8-Web-Hosting/S3-web-5.png)
---

### Bước 4: Chuẩn bị mã nguồn `index.html` hoàn chỉnh

1. Mở **Visual Studio Code** (hoặc trình soạn thảo bất kỳ) trên máy tính của bạn.
2. Mở file `index.html` trong thư mục dự án đã chuẩn bị ở Mục 5.2.
3. Thay thế toàn bộ nội dung file bằng đoạn mã hoàn chỉnh dưới đây:

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AWS Media Vault - Portal</title>
  <style>
    * { box-sizing: border-box; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
      max-width: 700px;
      margin: 40px auto;
      padding: 0 20px;
      background-color: #f4f6f8;
      color: #161e2e;
    }
    .header {
      text-align: center;
      margin-bottom: 30px;
    }
    .header h1 {
      margin: 0;
      color: #232f3e;
      font-size: 26px;
    }
    .header p {
      color: #68707f;
      margin-top: 8px;
    }
    .card {
      background: #ffffff;
      padding: 24px;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.08);
      margin-bottom: 24px;
      border: 1px solid #e1e4ea;
    }
    .card h2 {
      margin-top: 0;
      font-size: 18px;
      color: #232f3e;
      border-bottom: 2px solid #f4f6f8;
      padding-bottom: 10px;
    }
    .form-group {
      margin-top: 15px;
    }
    label {
      display: block;
      margin-bottom: 6px;
      font-weight: 600;
      font-size: 14px;
    }
    input[type="file"], input[type="text"] {
      width: 100%;
      padding: 10px;
      border: 1px solid #d5dbdb;
      border-radius: 4px;
      font-size: 14px;
      background: #fafafa;
    }
    input[type="text"]:focus {
      outline: none;
      border-color: #ec7211;
      background: #ffffff;
    }
    button {
      display: inline-block;
      width: 100%;
      padding: 12px;
      margin-top: 15px;
      border: none;
      border-radius: 4px;
      font-size: 15px;
      font-weight: bold;
      cursor: pointer;
      transition: background 0.2s ease-in-out;
    }
    .btn-upload {
      background-color: #ec7211;
      color: #ffffff;
    }
    .btn-upload:hover {
      background-color: #eb5f07;
    }
    .btn-download {
      background-color: #161e2e;
      color: #ffffff;
    }
    .btn-download:hover {
      background-color: #0a0f18;
    }
    .status {
      margin-top: 15px;
      padding: 12px;
      border-radius: 4px;
      font-size: 13px;
      line-height: 1.5;
      display: none;
      white-space: pre-wrap;
      word-break: break-all;
    }
    .status.info {
      background-color: #ebf8ff;
      border: 1px solid #bee3f8;
      color: #2b6cb0;
    }
    .status.success {
      background-color: #f0fff4;
      border: 1px solid #c6f6d5;
      color: #22543d;
    }
    .status.error {
      background-color: #fff5f5;
      border: 1px solid #fed7d7;
      color: #9b2c2c;
    }
    .status a {
      color: #2b6cb0;
      font-weight: bold;
      text-decoration: underline;
    }
  </style>
</head>
<body>

  <div class="header">
    <h1>AWS Media Vault Portal</h1>
    <p>Hệ thống lưu trữ, phân tích sự kiện & tải tệp an toàn trên AWS Serverless</p>
  </div>

  <!-- PHẦN 1: TẢI LÊN TỆP -->
  <div class="card">
    <h2>1. Tải lên Ảnh / Video (Upload)</h2>
    <div class="form-group">
      <label for="fileInput">Chọn tệp tin từ máy tính:</label>
      <input type="file" id="fileInput" accept="image/*,video/*">
    </div>
    <button class="btn-upload" onclick="uploadMedia()">Tải lên S3 Cloud</button>
    <div id="uploadStatus" class="status"></div>
  </div>

  <!-- PHẦN 2: TẢI XUỐNG TỆP -->
  <div class="card">
    <h2>2. Tải xuống tệp an toàn (Download)</h2>
    <div class="form-group">
      <label for="downloadFileName">Nhập tên tệp tin cần tải về:</label>
      <input type="text" id="downloadFileName" placeholder="Ví dụ: test.png hoặc video.mp4">
    </div>
    <button class="btn-download" onclick="downloadMedia()">Tạo liên kết tải an toàn</button>
    <div id="downloadStatus" class="status"></div>
  </div>

  <script>
    // =========================================================================
    // CẤU HÌNH ĐƯỜNG DẪN API GATEWAY CỦA BẠN (Lưu ý: Phải có /media ở cuối)
    // =========================================================================
    const API_ENDPOINT = "[https://xxxxxx.execute-api.ap-southeast-2.amazonaws.com/prod/media](https://xxxxxx.execute-api.ap-southeast-2.amazonaws.com/prod/media)";

    async function uploadMedia() {
      const fileInput = document.getElementById('fileInput');
      const status = document.getElementById('uploadStatus');

      if (!fileInput.files.length) {
        alert('Vui lòng chọn một tệp hình ảnh hoặc video trước khi tải lên!');
        return;
      }

      const file = fileInput.files[0];
      status.style.display = 'block';
      status.className = 'status info';
      status.innerText = `[1/2] Đang yêu cầu cấp quyền S3 Presigned URL cho tệp "${file.name}"...`;

      try {
        // Bước 1: Gọi API Gateway lấy Presigned URL (PUT)
        const response = await fetch(API_ENDPOINT, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            action: 'get_upload_url',
            fileName: file.name
          })
        });

        if (!response.ok) {
          throw new Error(`API Gateway từ chối kết nối (Mã lỗi: HTTP ${response.status})`);
        }

        const data = await response.json();
        if (!data.uploadUrl) {
          throw new Error('Không nhận được S3 Upload URL hợp lệ từ máy chủ.');
        }

        // Bước 2: Dùng Presigned URL đẩy file nhị phân trực tiếp lên S3
        status.innerText = `[2/2] Đang truyền dữ liệu nhị phân thẳng lên Amazon S3 Data Bucket...`;
        const uploadResponse = await fetch(data.uploadUrl, {
          method: 'PUT',
          body: file
        });

        if (uploadResponse.ok) {
          status.className = 'status success';
          status.innerText = `Tải lên thành công!\n\n` +
            `• Tên tệp: ${file.name}\n` +
            `• Kích thước: ${(file.size / 1024).toFixed(2)} KB\n` +
            `• S3 Trigger: Đã tự động kích hoạt Lambda trích xuất Metadata vào DynamoDB.\n` +
            `• Thông báo: Vui lòng kiểm tra hộp thư email cá nhân để xem bản tin chi tiết từ Amazon SNS.`;
        } else {
          throw new Error(`Amazon S3 từ chối tiếp nhận tệp (Mã phản hồi: HTTP ${uploadResponse.status})`);
        }
      } catch (error) {
        status.className = 'status error';
        status.innerText = `Xảy ra lỗi trong quá trình xử lý:\n${error.message}`;
      }
    }

    async function downloadMedia() {
      const fileNameInput = document.getElementById('downloadFileName');
      const status = document.getElementById('downloadStatus');
      const fileName = fileNameInput.value.trim();

      if (!fileName) {
        alert('Vui lòng nhập tên tệp tin cần tải về!');
        return;
      }

      status.style.display = 'block';
      status.className = 'status info';
      status.innerText = `Đang yêu cầu hệ thống sinh liên kết truy xuất cho tệp "${fileName}"...`;

      try {
        // Gọi API Gateway lấy Presigned URL (GET)
        const response = await fetch(API_ENDPOINT, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            action: 'get_download_url',
            fileName: fileName
          })
        });

        if (!response.ok) {
          throw new Error(`API Gateway báo lỗi HTTP ${response.status}`);
        }

        const data = await response.json();
        if (!data.downloadUrl) {
          throw new Error('Không thể tạo liên kết tải về cho tệp tin này.');
        }

        status.className = 'status success';
        status.innerHTML = `Liên kết tải về an toàn đã sẵn sàng (Có hiệu lực trong 5 phút):<br><br>` +
          `<a href="${data.downloadUrl}" target="_blank" download>` +
          `[ Bấm vào đây để tải xuống: ${fileName} ]` +
          `</a>`;
      } catch (error) {
        status.className = 'status error';
        status.innerText = `Lỗi truy xuất:\n${error.message}`;
      }
    }
  </script>
</body>
</html>
```

4. **Thay thế URL API:** Tìm đến dòng khai báo biến `API_ENDPOINT` ở dòng 182, thay thế chuỗi mẫu bằng Invoke URL thực tế bạn đã copy ở Mục 5.7:
   ```javascript
   const API_ENDPOINT = "https://<api-id>[.execute-api.ap-southeast-2.amazonaws.com/prod/media](https://.execute-api.ap-southeast-2.amazonaws.com/prod/media)";
   ```
5. Nhấn **Ctrl + S** (hoặc `Cmd + S`) để lưu file.

---

### Bước 5: Upload `index.html` lên S3 Web Bucket

1. Quay lại AWS Console $\rightarrow$ Mở dịch vụ **Amazon S3**.
2. Nhấp chọn vào bucket **`fcaj-media-portal-web-2026`**.
3. Tại tab **Objects**, nhấn nút màu cam **Upload**.
4. Nhấn nút **Add files** $\rightarrow$ Tìm và chọn file `index.html` vừa lưu trên máy tính.
5. Cuộn xuống cuối trang và nhấn nút **Upload**.

**Checkpoint:** Màn hình hiển thị thông báo màu xanh `Upload succeeded`. File `index.html` xuất hiện trong danh sách Objects của bucket web.

---

### Bước 6: Kiểm tra hoạt động của Web Portal

1. Quay lại tab **Properties** của bucket `fcaj-media-portal-web-2026`.
2. Cuộn xuống mục **Static website hosting** ở cuối trang.
3. Nhấp trực tiếp vào đường link **Bucket website endpoint** (ví dụ: `http://fcaj-media-portal-web-2026.s3-website-ap-southeast-2.amazonaws.com`).
4. Giao diện **AWS Media Vault Portal** xuất hiện trực tiếp trên trình duyệt web.

**Checkpoint:** Trang web hiển thị đầy đủ giao diện với 2 khối chức năng: Upload và Download, không xuất hiện bất kỳ lỗi kết nối nào khi tải trang.

---

## 3. Kết quả mong đợi

- Khởi tạo thành công S3 Bucket `fcaj-media-portal-web-2026` được cấu hình Static Website Hosting tại vùng Sydney `ap-southeast-2`.
- Thiết lập hoàn chỉnh Bucket Policy cho phép người dùng Internet đọc tệp tĩnh công khai mà vẫn đảm bảo an toàn cho dữ liệu nhạy cảm ở S3 Data Bucket.
- Triển khai thành công giao diện web portal hoàn chỉnh với mã nguồn `index.html` tích hợp Fetch API gọi REST API Gateway.
- Hệ thống sẵn sàng cho quá trình thiết lập hệ thống giám sát và cảnh báo thời gian thực với Amazon CloudWatch ở Mục 5.9.