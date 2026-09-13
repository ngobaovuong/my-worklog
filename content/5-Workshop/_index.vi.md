---
title: "Workshop"
date: 2026-09-13
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Triển khai AWS Media Vault trên AWS

#### Tổng quan

Trong workshop này, chúng ta sẽ xây dựng và triển khai **AWS Media Vault** — một nền tảng lưu trữ, xử lý đa phương tiện và cảnh báo sự kiện tự động theo kiến trúc Cloud-Native Serverless hoàn toàn trên AWS.

Giải pháp sử dụng 7 dịch vụ AWS cốt lõi bao gồm: **Amazon S3** (Lưu trữ nhị phân riêng tư & Static Web Hosting), **AWS Lambda** (Xử lý tính toán không máy chủ), **Amazon API Gateway** (REST API an toàn với CORS), **Amazon DynamoDB** (Lưu trữ Metadata NoSQL tốc độ cao), **Amazon SNS** (Hệ thống thông báo đẩy qua Email), **Amazon CloudWatch** (Thu thập Logs, Metrics & Cảnh báo Alarm), và **AWS IAM** (Quản lý định danh và phân quyền tối thiểu Least Privilege).

Toàn bộ quy trình sẽ hướng dẫn bạn từ khâu chuẩn bị tài khoản, cấu hình phân quyền IAM, thiết lập lưu trữ và cơ sở dữ liệu, viết mã nguồn Lambda, dựng cổng API Gateway, triển khai giao diện người dùng trực tiếp trên S3, cấu hình hệ thống giám sát CloudWatch, cho đến kiểm thử tích hợp, kiểm thử bơm lỗi và dọn dẹp tài nguyên.

#### Nội dung chi tiết

1. [Tổng quan Workshop](5.1-Workshop-overview/)
2. [Điều kiện chuẩn bị](5.2-Prerequisite/)
3. [Cấu hình phân quyền IAM](5.3-IAM-Setup/)
4. [Thiết lập Lưu trữ & Cơ sở dữ liệu](5.4-Storage-Database/)
5. [Cấu hình Thông báo qua SNS](5.5-SNS-Notification/)
6. [Phát triển logic với AWS Lambda](5.6-Lambda-Compute/)
7. [Xây dựng cổng kết nối API Gateway](5.7-API-Gateway/)
8. [Triển khai Web Hosting trên S3](5.8-Web-Hosting/)
9. [Giám sát & Báo động với CloudWatch](5.9-Monitoring/)
10. [Kiểm thử hệ thống & Bơm lỗi](5.10-Testing/)
11. [Dọn dẹp tài nguyên](5.11-Cleanup/)