---
title : "Storage & Database Setup"
date : 2026-09-25
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

### Objectives

Initialize an absolutely secure **Amazon S3** binary object storage repository with Block Public Access enabled, set up **CORS** configuration to allow direct client-side uploads from web browsers, and build an **Amazon DynamoDB** NoSQL table to store complete audit metadata for media files.

---

## 1. Theoretical Foundation and Data Architecture

The data layer of the system is built following an architecture that completely decouples raw binary object storage from structured metadata storage:

- **Amazon S3 Data Bucket (Private Binary Storage):**
  - Dedicated to storing original images and videos uploaded by users.
  - Strictly adheres to security principles: **100% Block All Public Access enabled**. No file can be publicly accessed from the outside unless authorized via a valid Presigned URL.
- **Cross-Origin Resource Sharing (CORS) Mechanism on S3:**
  - When the client-side web application (running from another domain or from S3 Web Hosting) sends an HTTP `PUT` request carrying file data directly to the S3 domain (`https://fcaj-media-source-demo-2026.s3.ap-southeast-2.amazonaws.com`), the web browser performs a CORS policy check.
  - If the S3 bucket has not been configured with CORS to allow the `PUT` method and the `*` header, the browser will immediately abort the connection and throw the common error: `Failed to fetch`.
- **Amazon DynamoDB (High-Speed NoSQL Database):**
  - Manages file metadata with a response latency under 10 milliseconds.
  - **Composite Primary Key:**
    - **Partition Key (`FileId` - String):** Identifies the name or path of the file.
    - **Sort Key (`UploadTime` - String):** Standard ISO 8601 (UTC) upload timestamp, helping manage the audit history of the same file if uploaded multiple times.

---

## 2. Implementation Steps

### Step 1: Initialize the Amazon S3 Data Bucket

1. Sign in to the **AWS Management Console** and verify that the selected Region is **Asia Pacific (Sydney) ap-southeast-2**.
2. In the search bar, type `S3` and select the **S3** service.
3. On the main S3 dashboard, click the orange **Create bucket** button.
![S3](/images/5-Workshop/5.4-Storage-Database/S31.png)
4. Under **General configuration**:
   - **Bucket type**: Select **General purpose**.
   - **Bucket name**: Enter the exact bucket name:
     ```text
     fcaj-media-source-demo-2026
     ```
     *(Note: S3 bucket names are globally unique. If this name is already taken, you can append a suffix like `fcaj-media-source-demo-2026-yourname` and ensure this new name is synchronized into the Lambda code later).*
   - **AWS Region**: Select **Asia Pacific (Sydney) ap-southeast-2**.
5. Under **Object Ownership**: Keep the default **ACLs disabled (recommended)**.
6. Under **Block Public Access settings for this bucket**:
   - Keep **Block all public access** checked (all 4 checkboxes below must be checked).
7. For **Bucket Versioning**, **Tags**, and **Default encryption**: Keep the default settings (Amazon S3 managed keys - SSE-S3).
8. Scroll to the bottom of the page and click the **Create bucket** button.
![S32](/images/5-Workshop/5.4-Storage-Database/S32.png)
![S33](/images/5-Workshop/5.4-Storage-Database/S33.png)
![S34](/images/5-Workshop/5.4-Storage-Database/S34.png)
![S35](/images/5-Workshop/5.4-Storage-Database/S35.png)
**Checkpoint:** The bucket list displays `fcaj-media-source-demo-2026` with the status **Objects can be public: Bucket and objects not public** and the AWS Region as **ap-southeast-2**.

---

### Step 2: Configure CORS for the S3 Data Bucket

1. In the S3 bucket list, click the bucket name **`fcaj-media-source-demo-2026`**.
2. Switch to the **Permissions** tab.
3. Scroll down to the **Cross-origin resource sharing (CORS)** section near the bottom of the page $\rightarrow$ Click the **Edit** button.
4. In the JSON editor, paste the following complete CORS rule configuration:

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

5. Click the **Save changes** button.

**Checkpoint:** In the Cross-origin resource sharing (CORS) section, the JSON configuration displays the full methods `GET, PUT, POST, HEAD` with `AllowedOrigins: *`.

---

### Step 3: Create the Amazon DynamoDB Table

1. In the console search bar, type `DynamoDB` and select the **DynamoDB** service.
2. In the left navigation pane, select **Tables** $\rightarrow$ Click the orange **Create table** button.
![DynamoDB1](/images/5-Workshop/5.4-Storage-Database/DynamoDB1.png)
3. On the **Create DynamoDB table** configuration page:
   - **Table name**: Enter exactly:
     ```text
     MediaMetadata
     ```
   - **Partition key**: Enter `FileId` $\rightarrow$ Data type select **String**.
   - **Sort key**: Check the *Add sort key* box $\rightarrow$ Enter `UploadTime` $\rightarrow$ Data type select **String**.
4. Under **Table class**: Select **DynamoDB Standard**.
5. Under **Capacity specs**: Select **Default settings** (Automatically activates the Provisioned 5 RCU / 5 WCU tier, completely free under the AWS Free Tier limit).
6. Scroll to the bottom of the page and click the **Create table** button.
![DynamoDB2](/images/5-Workshop/5.4-Storage-Database/DynamoDB2.png)

```text
Table name: MediaMetadata
Partition key: FileId (String)
Sort key: UploadTime (String)
Table class: DynamoDB Standard
```

7. Wait about 10 to 15 seconds for the table to finish initializing.
![DynamoDB3](/images/5-Workshop/5.4-Storage-Database/DynamoDB3.png)

**Checkpoint:** The status of table `MediaMetadata` turns green as **Active**. Click the table name to verify the ARN details match the format: `arn:aws:dynamodb:ap-southeast-2:<account-id>:table/MediaMetadata`.

---

## 3. Expected Results

- Successfully created the S3 Data Bucket `fcaj-media-source-demo-2026` in the Sydney region, 100% secured against raw public object exposure.
- Fully configured CORS on S3, ready to receive uploaded files via HTTP `PUT` from any client browser using a Presigned URL.
- The DynamoDB NoSQL table `MediaMetadata` is running in the **Active** state, ready to receive metadata written by AWS Lambda.
- The storage and database infrastructure is ready to proceed to the Amazon SNS notification service configuration in the next chapter.