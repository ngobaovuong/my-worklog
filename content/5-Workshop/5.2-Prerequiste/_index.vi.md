---
title : "Điều kiện chuẩn bị"
date : 2026-09-25
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

### Mục tiêu

Đảm bảo người đọc hoàn thiện việc đăng ký tài khoản AWS Free Tier, thiết lập bảo mật cơ bản, xác định đúng Region hoạt động, chuẩn bị đầy đủ các công cụ phần mềm trên máy cục bộ và tải mã nguồn dự án trước khi bắt đầu triển khai hệ thống **AWS Media Vault**.

---

## 1. Đăng ký và chuẩn bị tài khoản AWS

Workshop này sử dụng các dịch vụ hoàn toàn nằm trong gói **AWS Free Tier**. Nếu chưa có tài khoản AWS, bạn cần chuẩn bị:

- **Thẻ thanh toán quốc tế (Visa/Mastercard)**: Có sẵn ít nhất 1 USD (khoảng 25.000 VNĐ) để AWS xác thực danh tính thẻ (khoản này sẽ được hoàn trả lại tự động).
- **Số điện thoại & Email cá nhân**: Dùng để nhận mã xác minh OTP khi đăng ký và dùng làm kênh nhận thông báo cảnh báo sự cố từ Amazon SNS.

### Các bước đăng ký tài khoản AWS:

1. Truy cập trang chủ đăng ký: [https://aws.amazon.com/free/](https://aws.amazon.com/free/) và chọn **Create a Free Account**.
2. Nhập địa chỉ Email gốc (Root user email) và đặt tên cho tài khoản (AWS account name).
3. Xác minh địa chỉ email qua mã OTP gửi về hòm thư.
4. Thiết lập mật khẩu mạnh cho tài khoản Root (tối thiểu 8 ký tự, gồm chữ hoa, chữ thường, số và ký tự đặc biệt).
5. Điền thông tin liên lạc cá nhân (chọn loại tài khoản **Personal**).
6. Nhập thông tin thẻ thanh toán quốc tế (Số thẻ, ngày hết hạn, tên in trên thẻ, mã CVV/CVC).
7. Xác minh danh tính số điện thoại qua tin nhắn SMS chứa mã kích hoạt.
8. Tại bước chọn gói hỗ trợ (Support Plan), chọn gói **Basic Support - Free** (Miễn phí).
9. Đăng nhập vào AWS Management Console sau khi tài khoản được kích hoạt thành công.
![Trang chủ đăng ký](/images/2-Proposal/proposal1.png)

---

## 2. Công cụ và môi trường cục bộ cần chuẩn bị

Hệ thống được thiết kế theo mô hình Serverless và thao tác trực tiếp trên **AWS Management Console (Web UI)** mà không đòi hỏi cài đặt các bộ công cụ IaC phức tạp.

Chuẩn bị các phần mềm sau trên máy tính cá nhân:

- **Trình duyệt web hiện đại**: Google Chrome, Mozilla Firefox hoặc Microsoft Edge (cập nhật bản mới nhất để không gặp lỗi hiển thị Console và hỗ trợ kiểm thử tính năng Fetch API trên web portal).
- **Visual Studio Code (hoặc IDE khác)**: Dùng để chỉnh sửa đường dẫn API trong file `index.html` và quản lý mã nguồn `lambda_function.py`.
- **Git**: Dùng để clone mã nguồn mẫu từ GitHub và quản lý phiên bản.
- **Python (v3.10+)**: Dùng để kiểm tra cú pháp mã nguồn Lambda hoặc kiểm thử SDK Boto3 trên máy (nếu cần).
- **Hòm thư Email cá nhân**: Dùng để nhận liên kết xác thực nhận tin từ Amazon SNS (Subscription Confirmation).

---

## 3. Các bước thực hiện

**Đăng nhập AWS Console:** Đăng nhập vào AWS Management Console bằng tài khoản vừa chuẩn bị. Đảm bảo Region hiển thị ở góc trên bên phải màn hình được chuyển chính xác sang **ap-southeast-2 (Sydney)**.
![Trang AWS Console](/images/2-Proposal/proposal2.png)

**Checkpoint:** Xác nhận Region hiển thị là **Asia Pacific (Sydney) ap-southeast-2** trước khi khởi tạo bất kỳ tài nguyên nào để đảm bảo tính tương thích giữa S3, Lambda, API Gateway và DynamoDB.

**Kiểm tra công cụ cục bộ:** Mở Terminal (trên macOS/Linux) hoặc Command Prompt / PowerShell (trên Windows) để kiểm tra các phần mềm cơ bản đã sẵn sàng:

```bash
git --version
python --version   # hoặc python3 --version
code --version     # kiểm tra Visual Studio Code (tùy chọn)
```

**Checkpoint:** Tất cả các lệnh đều trả về phiên bản hợp lệ của Git và Python.

**Tải và chuẩn bị mã nguồn dự án:** Clone mã nguồn dự án từ kho lưu trữ GitHub về máy hoặc tạo một thư mục làm việc mới:

```bash
git clone ...
cd aws-media-vault
code .
```

Cấu trúc thư mục dự án cần có các tệp cơ bản sau:
- `index.html`: Giao diện người dùng web portal để tải lên và tải xuống tệp media trực tiếp qua S3 Presigned URL.
- `lambda_function.py`: Mã nguồn logic của AWS Lambda viết bằng Python 3.12 sử dụng Boto3.

**Checkpoint:** Thư mục dự án được mở thành công trong Visual Studio Code, mã nguồn và môi trường đã sẵn sàng cho bước cấu hình bảo mật IAM ở chương tiếp theo.

---

## 4. Kết quả mong đợi

- Đăng ký và đăng nhập thành công vào AWS Management Console với gói Free Tier tại Region **ap-southeast-2 (Sydney)**.
- Chuẩn bị đầy đủ môi trường phát triển cục bộ (Trình duyệt, VS Code, Git, Python).
- Sẵn sàng hòm thư email để nhận các thông báo đẩy từ AWS.
- Tải thành công mã nguồn dự án và hiểu rõ cấu trúc các tệp tin trước khi bắt đầu triển khai hạ tầng.