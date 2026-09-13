---
title : "Workshop Overview"
date : 2026-09-13
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

### Objectives

This workshop guides you through deploying the **AWS Media Vault** application on AWS using a Serverless Cloud-Native architecture, AWS Managed Services, Event-Driven processing, and secure file transfers via S3 Presigned URLs. Upon completion of this workshop, you will be able to deploy a production-ready media processing and storage platform featuring instant scalability, zero idle costs, and enterprise-grade security adhering to the AWS Well-Architected Framework.

---

## 1. Problem Statement & Solution

**AWS Media Vault** is a web-based platform allowing users to securely upload, store, and download multimedia assets (images and videos). The system supports direct browser uploads, automated metadata extraction, NoSQL audit logging, instant email reporting, and real-time operational failure tracking.

Instead of running the application on traditional virtual machines (such as Amazon EC2) which incur ongoing idle costs and storage security risks, this workshop implements a 100% serverless architecture on AWS. The client frontend is served statically via **Amazon S3 Static Website Hosting**. The browser interfaces securely with cloud backend logic through **Amazon API Gateway** and **AWS Lambda** to synthesize **S3 Presigned URLs**, allowing direct, authenticated uploads into private **Amazon S3** buckets without opening public bucket access or exposing AWS IAM credentials.

To enhance automation and observability, metadata for every uploaded object is stored in **Amazon DynamoDB**, while formatted event digests are fanned out via **Amazon SNS** to administrator inboxes. Inter-service privileges are tightly governed by **AWS IAM** adhering to the Principle of Least Privilege. Simultaneously, **Amazon CloudWatch** aggregates execution logs, tracks runtime metrics, and triggers alarms upon application errors.

---

## 2. System Architecture

The overall system architecture comprises the following core components:

- End User (Web Browser Client)
- Static Presentation Layer (Amazon S3 Website Hosting)
- Managed API Entry Point with CORS (Amazon API Gateway)
- Serverless Compute Layer (AWS Lambda)
- Private Object Storage (Amazon S3 Data Bucket)
- NoSQL Database Layer (Amazon DynamoDB)
- Push Alerting Engine (Amazon SNS)
- Real-time Observability & Alarms (Amazon CloudWatch)

**Figure 1 – AWS Media Vault System Architecture**

![System Architecture](/images/5-Workshop/5.1-Workshop-overview/diagram.drawio.png)

---

## 3. System Workflow

The primary execution workflow proceeds through the following sequential stages:

1. The client accesses the web application hosted via **Amazon S3 Static Website Hosting**.

2. Upon selecting a media file to upload, the browser issues an asynchronous HTTP POST request to **Amazon API Gateway** on the CORS-enabled `/media` endpoint.

3. API Gateway transparently proxies the payload to **AWS Lambda** via Lambda Proxy Integration.

4. The Lambda function uses the AWS SDK (Boto3) to generate an ephemeral, SigV4-signed **S3 Presigned URL (PUT)** with a 300-second expiration and returns it to the client.

5. The browser issues a direct HTTP PUT request using the Presigned URL to push the binary file directly into the private **Amazon S3 (Data Bucket)**.

6. The arrival of the object emits an `s3:ObjectCreated:*` event, which automatically triggers the **AWS Lambda** backend processing flow.

7. Lambda extracts the file metadata (key name, size, MIME type, upload timestamp) and writes an audit record into the **Amazon DynamoDB** table.

8. Lambda formats a structured transaction digest and publishes the payload to an **Amazon SNS Topic**, which automatically dispatches an alert email to the administrator.

9. Execution logs and telemetry metrics are streamed to **Amazon CloudWatch**. If an unhandled exception occurs, a CloudWatch Alarm triggers an urgent incident notification via SNS.

10. When a user requests a file download, Lambda generates a temporary **S3 Presigned URL (GET)**, allowing secure retrieval from S3 without exposing internal bucket paths.

---

## 4. AWS Services Utilized

This workshop leverages the following AWS services:

### Frontend & API Ingress

- Amazon S3 (Static Website Hosting)
- Amazon API Gateway (REST API & CORS)

### Serverless Compute

- AWS Lambda (Runtime Python 3.12)

### Storage & Database

- Amazon S3 (Data Bucket)
- Amazon DynamoDB

### Messaging & Notifications

- Amazon Simple Notification Service (Amazon SNS)

### Security & Access Control

- AWS Identity and Access Management (AWS IAM)
- S3 Bucket Policies & CORS Configuration

### Observability & Operations

- Amazon CloudWatch Logs
- Amazon CloudWatch Metrics & Alarms

---

## 5. Learning Outcomes

Upon completing this workshop, you will be able to:

- Build and host a static frontend using Amazon S3 Static Website Hosting.
- Provision a REST API on Amazon API Gateway supporting CORS preflight checks for ANY and OPTIONS methods.
- Develop an AWS Lambda function (Python 3.12) capable of handling both synchronous HTTP requests and asynchronous S3 event records.
- Implement and master AWS SigV4 S3 Presigned URL generation for direct PUT/GET transfers.
- Configure Cross-Origin Resource Sharing (CORS) rules on private S3 buckets for client-side uploads.
- Persist and query NoSQL metadata records in Amazon DynamoDB.
- Set up an automated notification pipeline with Amazon SNS Topics and Email Subscriptions.
- Enforce IAM Least Privilege access while diagnosing and resolving Permissions Boundary restrictions.
- Centralize operational logging and configure proactive CloudWatch Alarms for error states.
- Perform end-to-end functional validation, download verification, and fault-injection testing.
- Tear down all provisioned cloud assets to guarantee zero ongoing billing post-workshop.