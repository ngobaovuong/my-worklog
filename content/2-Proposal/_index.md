---
title: "Proposal"
date: 2026-09-13
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AWS Media Vault

## Serverless Media Storage & Monitoring Platform on AWS

---

# 1. Executive Summary

AWS Media Vault is a cloud-native, serverless solution designed to provide secure, automated storage, processing, and retrieval of rich media files (images and videos). The platform delivers an intuitive web portal, automated metadata extraction, audit log persistence, real-time email alerting, and operational error tracking—relying entirely on AWS managed services to minimize costs, guarantee high availability, and eliminate server maintenance overhead.

The client-side interface is built with **HTML5**, **JavaScript (ES6+)**, and **CSS3**, hosted statically on **Amazon S3 (Static Website Hosting)**. The frontend securely interfaces with cloud infrastructure via **Amazon API Gateway** and **AWS Lambda** utilizing **S3 Presigned URLs**, allowing users to upload data directly to private storage buckets without exposing AWS credentials or requiring public bucket permissions.

The backend infrastructure leverages **AWS Lambda** (Python 3.12) as an event-driven compute engine. Object metadata is synchronized into **Amazon DynamoDB**, detailed processing reports are published via **Amazon SNS** to administrative inboxes, execution metrics and diagnostic logs are collected by **Amazon CloudWatch**, and access boundaries are enforced through **AWS IAM** following the principle of least privilege. This architecture delivers an automated, highly secure, zero-idle-cost media pipeline built for instant scalability.

---

# 2. Problem Statement

## Current Challenges

Traditional media storage and management applications typically rely on persistent virtual instances (such as Amazon EC2 or conventional VPS), introducing notable architectural drawbacks:

- **Idle Resource Costs:** Servers must operate and incur billing 24/7 regardless of actual upload traffic.
- **Storage Security Vulnerabilities:** Enabling uploads frequently leads administrators to grant public bucket access or hardcode AWS Access Keys into frontend source code.
- **Compute Bottlenecks:** Web servers act as intermediary proxies for bulky media uploads, resulting in heavy memory and CPU contention.
- **Lack of Real-time Visibility:** Administrators lack immediate notifications when critical files are processed or when processing pipelines experience failure.

## Proposed Solution

The proposed solution implements a fully serverless, event-driven architecture using managed AWS services:

- Users access a responsive web portal served via **Amazon S3 Website Hosting** to initiate secure upload and download requests.
- **Amazon API Gateway** receives client requests and triggers **AWS Lambda** to generate time-limited **S3 Presigned URLs**, enabling direct upload into private **Amazon S3 Data Buckets**.
- An `s3:ObjectCreated:*` event automatically triggers Lambda to extract object metadata, persist audit logs to **Amazon DynamoDB**, and dispatch formatted notifications via **Amazon SNS**.
- Pipeline execution health and error metrics are continually tracked by **Amazon CloudWatch Metric Filters & Alarms**.

## Key Benefits

The proposed architecture delivers substantial technical and business advantages:

- **Zero Idle Cost:** Billing is strictly tied to millisecond compute intervals and actual transactions within the AWS Free Tier.
- **Robust Security Posture:** Private data buckets block all public access; data exchange relies entirely on ephemeral digital signatures.
- **Effortless Scalability:** Decoupled serverless services scale seamlessly from a single file to millions of concurrent requests.
- **Operational Simplicity:** Eliminates operating system patching, complex network topology maintenance, and server management.
- **End-to-End Observability:** Transparent structured logging and automated email alerts on operational failures.

---

# 3. Solution Architecture

The solution implements an event-driven serverless architecture, cleanly separating static frontend distribution from dynamic backend event processing.

## Architecture Diagram

![System Architecture](/images/2-Proposal/system_architecture.png)

## AWS Services Utilized

- **Amazon S3 (Hosting Bucket):** Static web hosting for frontend client files.
- **Amazon S3 (Data Bucket):** Secure, private storage for uploaded media objects.
- **Amazon API Gateway:** Managed REST API endpoint handling client-to-cloud requests.
- **AWS Lambda:** Serverless compute generating presigned URLs and handling S3 events.
- **Amazon DynamoDB:** Managed NoSQL database storing media metadata and audit trails.
- **Amazon SNS (Simple Notification Service):** Pub/Sub alerting engine dispatching email updates.
- **Amazon CloudWatch:** Comprehensive log aggregation, operational metrics, and error alarms.
- **AWS IAM:** Identity and fine-grained access control enforcing Least Privilege.

## Component Design

### Frontend

- HTML5 / CSS3
- Vanilla JavaScript (Fetch API, ES6+)
- Hosted on S3 Static Website Hosting

### Backend & API Layer

- Amazon API Gateway (REST API, CORS Enabled)
- AWS Lambda (Python 3.12 Runtime)
- AWS SDK for Python (Boto3)

### Database Layer

- Amazon DynamoDB (On-Demand / Provisioned Capacity)

### Storage Layer

- Amazon S3 Standard (CORS Enabled, SigV4 Signature Version)

### Monitoring & Notification Layer

- Amazon SNS Topic (Email Protocol)
- Amazon CloudWatch Log Groups
- Amazon CloudWatch Alarm (Metric `Errors` $\ge 1$)

### Data Flow Workflow

Web Browser

↓ *(1. Request Presigned URL via REST API)*

Amazon API Gateway

↓ *(2. Invoke function for ephemeral credentials)*

AWS Lambda

↓ *(3. Direct HTTP PUT upload)*

Amazon S3 (Data Bucket)

↓ *(4. Emit s3:ObjectCreated event)*

AWS Lambda

├── *(5a. Persist Metadata)* ──> Amazon DynamoDB

├── *(5b. Dispatch Notifications)* ──> Amazon SNS ──> Admin Inbox

└── *(5c. Export Diagnostics)* ──> Amazon CloudWatch

---

# 4. Technical Implementation

## Implementation Phases

The project was executed through the following structured milestones:

- Researched event-driven serverless design patterns and AWS Well-Architected standards.
- Designed system architecture and end-to-end data flow specifications.
- Provisioned private S3 data buckets and established Cross-Origin Resource Sharing (CORS) rules.
- Created the NoSQL data schema on Amazon DynamoDB.
- Configured the Amazon SNS alerting topic and confirmed email subscriptions.
- Authored fine-grained IAM execution policies adhering strictly to least privilege access.
- Implemented multi-purpose Lambda handlers in Python 3.12 for API Gateway integration and S3 event routing.
- Deployed Amazon API Gateway REST endpoints with custom CORS definitions to the `prod` stage.
- Built a lightweight web interface and deployed static assets via S3 Website Hosting.
- Created CloudWatch Log Groups and established proactive alarm thresholds.
- Conducted comprehensive functional testing (Happy Path, Error Scenarios, Fault Injection).
- Validated cost controls and compiled complete resource clean-up procedures.

## Technical Requirements

### Programming Languages

- Python 3.12
- JavaScript (Vanilla ES6)
- HTML5 / CSS3

### Core Libraries & SDKs

- Boto3 (AWS SDK for Python)
- Botocore (Config Signature Version S3v4)
- Postman / cURL (API Testing)

### Cloud Infrastructure (AWS)

- Amazon S3
- Amazon API Gateway
- AWS Lambda
- Amazon DynamoDB
- Amazon SNS
- Amazon CloudWatch
- AWS IAM

### Development Tools

- Visual Studio Code
- Git & GitHub
- AWS Management Console
- Draw.io / Excalidraw

---

# 5. Project Roadmap & Milestones

The project was delivered across the following structured timeline:

### Phase 1 – Architecture & Planning

- Analyzed serverless processing requirements and edge cases.
- Drafted system architecture diagrams and operational workflows.
- Defined the IAM least-privilege matrix across all services.

### Phase 2 – Backend Core Infrastructure

- Created the S3 Data Bucket with granular CORS rules.
- Provisioned the `MediaMetadata` DynamoDB table with `FileId` partition key.
- Created the `MediaProcessingAlerts` SNS topic and verified subscriber subscriptions.

### Phase 3 – Compute Engine Development

- Configured the IAM Lambda execution role, ensuring no restrictive boundary conflicts.
- Authored the dual-action Lambda function (Presigned URL generator & S3 event handler).
- Executed unit tests in the AWS Lambda console using synthetic `s3-put` events.

### Phase 4 – API & Web Portal Integration

- Built the REST API Gateway endpoint `/media` with `ANY` method support.
- Enabled CORS across methods and deployed the API to the `prod` stage.
- Developed `index.html` featuring asynchronous upload and download functionality.
- Published web assets through S3 Static Website Hosting.

### Phase 5 – Monitoring & Observability

- Attached `AWSLambdaBasicExecutionRole` for automated CloudWatch Log Group creation.
- Configured a CloudWatch Alarm evaluating Lambda `Errors` ($\ge 1$ threshold).
- Connected alarm trigger actions to the designated SNS Topic.

### Phase 6 – Verification & Fault Injection Testing

- Validated successful upload flows: Client $\rightarrow$ S3 $\rightarrow$ DynamoDB $\rightarrow$ SNS email delivery.
- Conducted fault-injection tests (revoking permissions) to verify CloudWatch alarms and email alerts.
- Verified download functionality via 5-minute expiring Presigned GET URLs.

### Phase 7 – Final Documentation & Packaging

- Authored bilingual (VI/EN) technical workshop documentation.
- Packaged complete source code, IAM templates, and assets to GitHub.
- Finalized project summary presentation.

---

# 6. Cost Estimation

## Infrastructure Cost Analysis

The solution is architected entirely on serverless primitives, falling fully within the **AWS Free Tier**:

| AWS Service | Free Tier Allowance | Anticipated Monthly Usage | Estimated Cost |
|---|---|---|---|
| **Amazon S3** | 5 GB Standard Storage, 20,000 GET, 2,000 PUT | ~200 MB, ~500 requests | $0.00 / month |
| **AWS Lambda** | 1,000,000 free requests, 3.2M sec compute | ~1,000 invocations | $0.00 / month |
| **Amazon API Gateway** | 1,000,000 REST API calls/month (First 12 mo.) | ~1,500 calls | $0.00 / month |
| **Amazon DynamoDB** | 25 GB storage, 25 WCU / 25 RCU capacity | < 10 MB, 5 WCU / 5 RCU | $0.00 / month |
| **Amazon SNS** | 1,000 email notifications/month | ~100 emails | $0.00 / month |
| **Amazon CloudWatch** | 10 custom metrics, 10 alarms, 5 GB log data | 1 Alarm, 1 Log Group (~50 MB) | $0.00 / month |
| **AWS IAM** | Free service tier | Unlimited roles/policies | $0.00 / month |
| **Total Estimated Cost** | | | **~$0.00 / month** |

### Cost Optimization Guidelines

- **AWS Budgets:** Configured zero-spend threshold alarms notifying administrators if expenses exceed **$1.00**.
- **Presigned URL Expiry:** Configured tight token lifetimes (**300 seconds / 5 minutes**) to prevent unauthorized link reuse.
- **S3 Lifecycle Rules:** Configured automated lifecycle policies transitioning demo objects or purging temporary files after 7 days.
- **Clean-up Procedures:** Detailed resource tear-down scripts provided to delete test buckets, API definitions, tables, alarms, and functions upon evaluation completion.

---

# 7. Risk Assessment

## Risk Matrix

- IAM Access Denied errors stemming from unintentional Permissions Boundary constraints.
- Browser CORS failures during preflight OPTIONS requests to API Gateway or S3.
- Recursive Lambda invocations if output files write back to the event trigger bucket prefix.
- Accidental credential exposure within frontend application scripts.
- Silent execution failures due to missing CloudWatch logging permissions.
- Uncontrolled budget spikes caused by large payload abuse.

## Mitigation Strategies

- Audited IAM roles to remove unnecessary boundaries and enforce strict least-privilege scoping.
- Standardized CORS configurations across API Gateway resources and S3 bucket definitions.
- Isolated static hosting buckets completely from data storage buckets to eliminate invocation loops.
- Adopted transient Presigned URLs generated server-side, eliminating long-term credentials in clients.
- Attached `AWSLambdaBasicExecutionRole` explicitly to guarantee continuous log streaming.
- Enforced client-side payload validation and enabled CloudWatch billing threshold notifications.

## Contingency Plans

- **Permission Denials:** Inspect CloudWatch Log Streams or execute local Lambda mock tests to parse explicit `AccessDeniedException` error traces.
- **API Connectivity Failures:** Run direct cURL requests against API endpoints to isolate frontend CORS issues from backend logic errors.
- **Alarm Triggers:** Access `/aws/lambda/process-media-metadata` log groups immediately to evaluate stack traces and deploy hotfixes.
- **Unhandled Exceptions:** Implement defensive `try...except` exception blocks and route formatted error diagnostics directly via SNS.

---

# 8. Expected Outcomes

## Technical Deliverables

Upon completion, the project delivers:

- A functional, responsive web portal supporting direct media upload and secure file retrieval.
- A 100% serverless, event-driven data processing pipeline requiring zero server maintenance.
- Secure, short-lived S3 Presigned URL data transfers signed with AWS SigV4.
- Centralized metadata logging inside a scalable DynamoDB NoSQL table.
- An automated real-time notification engine dispatching structured media summaries via Amazon SNS.
- Continuous observability powered by CloudWatch Log Groups, metrics, and error alarms.
- Comprehensive, bilingual workshop documentation enabling end-to-end reproducibility.

## Business Value

The project demonstrates practical implementation of key pillars from the AWS Well-Architected Framework:

- **Cost Optimization:** Achieved 100% elimination of idle infrastructure costs compared to conventional server deployments.
- **Operational Excellence:** Fully automated end-to-end lifecycle from upload ingestion to operational failure alerting.
- **Security & Reliability:** Zero public data access, least-privilege role scoping, and high durability backed by Amazon S3's 99.999999999% (11 9's) architecture.

Future enhancements may introduce **AWS Rekognition** for automated content moderation and computer vision labeling, **Amazon CloudFront** for global edge caching, and **Amazon Cognito** for multi-tenant user authentication.