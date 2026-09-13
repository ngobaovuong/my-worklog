---
title : "System Testing & Fault Injection"
date : 2026-09-13
weight : 10
chapter : false
pre : " <b> 5.10. </b> "
---

### Objectives

Execute comprehensive End-to-End (E2E) integration testing for the **AWS Media Vault** platform to validate data integrity across client and serverless infrastructure layers. Conduct intentional **Fault Injection** to verify granular runtime exception logging in Amazon CloudWatch and validate proactive incident alarming and automated email dispatching via Amazon SNS.

---

## 1. Theoretical Concepts & Testing Methodology

To satisfy AWS Well-Architected operational standards, system validation spans two core operational tracks:

### 1. Happy Path Pipeline Validation:
- **Direct Upload Pipeline:**
  1. Client browser issues an HTTP POST request to API Gateway (`/media`) requesting an S3 Presigned URL.
  2. API Gateway invokes Lambda, which uses Boto3 to generate a SigV4-signed URL valid for 300 seconds.
  3. Client uses the Presigned URL to stream the binary payload directly into S3 Data Bucket `fcaj-media-source-demo-2026` via HTTP `PUT`.
  4. S3 emits an `s3:ObjectCreated:*` asynchronous trigger to Lambda.
  5. Lambda resolves safe URL encodings with `urllib.parse.unquote_plus`, parses extensions, and calculates size metrics in KB.
  6. Lambda persists audit records into DynamoDB table `MediaMetadata`.
  7. Lambda publishes a formatted notification digest to SNS Topic `MediaProcessingAlerts` dispatching emails to administrators.
- **Secure Retrieval (Download) Pipeline:**
  1. Client sends target object key to API Gateway.
  2. Lambda generates a short-lived Presigned URL `GET` valid for 5 minutes.
  3. Client downloads the binary file directly from private S3 storage without exposing internal bucket configurations.

### 2. Fault Injection & Incident Response Verification:
- Deliberately alter the IAM execution role permissions by stripping `dynamodb:PutItem` (reproducing the `AccessDeniedException` and Permissions Boundary conflicts examined in Section 5.3).
- **System Failure Telemetry Verification:**
  - Client uploads to S3 succeed, but Lambda backend execution triggers an unhandled `ClientError` exception.
  - CloudWatch Logs captures complete Boto3 SDK stack traces.
  - Lambda `Errors` metric increments to $\ge 1$.
  - CloudWatch Alarm `MediaLambdaFailureAlarm` immediately switches state to **In alarm** (red).
  - Amazon SNS dispatches an automated incident email alert to administrator inboxes.

---

## 2. Step-by-Step Instructions

### Scenario 1: Happy Path Pipeline Validation

#### Step 1: Access Web Portal via S3 Website Endpoint
1. Launch a modern web browser (Google Chrome or Mozilla Firefox).
2. Paste the **Bucket website endpoint** retrieved in Section 5.8:
   ```text
   [http://fcaj-media-portal-web-2026.s3-website-ap-southeast-2.amazonaws.com](http://fcaj-media-portal-web-2026.s3-website-ap-southeast-2.amazonaws.com)
   ```
3. Press **F12** to open browser **Developer Tools** $\rightarrow$ Select the **Console** and **Network** tabs to observe API handshakes.
![Test1](/images/5-Workshop/5.10-Testing/Test1.png)
#### Step 2: Upload Media File
1. Under **1. Upload Media (Images / Videos)**, click the file input button.
2. Select an image or video file from your computer (e.g., `meomeo.jfif` or `demo.png`).
3. Click the orange **Upload to S3 Cloud** button.
4. Monitor UI feedback:
   - Progress transitions from blue (*Requesting Presigned URL...*) to green success status:
     ```text
     Upload Succeeded!
     • Filename: meomeo.jfif
     • Payload Size: 23.50 KB
     • S3 Event: Automatically triggered Lambda metadata extraction to DynamoDB.
     • Alert: Check your email inbox for the Amazon SNS delivery summary.
     ```
![Test2](/images/5-Workshop/5.10-Testing/Test2.png)
**Checkpoint:** The Web Portal presents a green success confirmation with zero console errors in the browser Developer Tools.

#### Step 3: Validate Object Presence in Amazon S3
1. Navigate to the **Amazon S3 Console**.
2. Open bucket **`fcaj-media-source-demo-2026`**.
3. Under the **Objects** tab, verify object presence:
   - `meomeo.jfif` displays with accurate size and upload timestamps.
![Test3](/images/5-Workshop/5.10-Testing/Test3.png)
#### Step 4: Validate Record Persistence in Amazon DynamoDB
1. Open the **Amazon DynamoDB Console** $\rightarrow$ Select **Tables** in the sidebar.
2. Click on the table name **`MediaMetadata`**.
3. Click the **Explore table items** button in the top-right corner.
4. Under **Items returned**, click the newly inserted record to verify attributes:
   - `FileId`: `meomeo.jfif`
   - `UploadTime`: ISO 8601 UTC timestamp (e.g., `2026-09-11 06:45:20 UTC`)
   - `Bucket`: `fcaj-media-source-demo-2026`
   - `Format`: `JFIF`
   - `SizeBytes`: `24064`
   - `Status`: `ACTIVE`
![Test4](/images/5-Workshop/5.10-Testing/Test4.png)
![Test5](/images/5-Workshop/5.10-Testing/Test5.png)
#### Step 5: Verify Amazon SNS Delivery Notification
1. Open your personal email inbox.
2. Locate the email from **AWS Notifications** titled:
   ```text
   [AWS Media Vault] File meomeo.jfif Processed Successfully
   ```
3. Verify the formatted ASCII message digest:
   ```text
   ===========================================
          AWS MEDIA VAULT - NEW UPLOAD        
   ===========================================
   • File Name      : meomeo.jfif
   • Extension      : JFIF
   • Size           : 24064 Bytes (~23.5 KB)
   • Target Bucket  : fcaj-media-source-demo-2026
   • Uploaded At    : 2026-09-11 06:45:20 UTC
   • Database Sync  : DynamoDB [MediaMetadata] - SUCCESS
   ===========================================
   Status: Encrypted and ready for secure retrieval.
   ```
![Test6](/images/5-Workshop/5.10-Testing/Test6.png)
#### Step 6: Test Secure Media Retrieval (Download)
1. Return to the Web Portal browser window.
2. Under **2. Secure Media Retrieval (Download)**:
   - Enter the exact filename: `meomeo.jfif`.
3. Click the dark button: **Generate Download Link**.
4. The green banner presents the temporary download link:
   ```text
   [ Click here to download: meomeo.jfif ]
   ```
5. Click the link: The browser opens or downloads `meomeo.jfif` directly from S3 using the SigV4 Presigned URL.
![Test7](/images/5-Workshop/5.10-Testing/Test7.png)
**Checkpoint:** Successful file download confirms end-to-end execution of the Happy Path workflow.

---

### Scenario 2: Fault Injection & Alarm Triggering

#### Step 1: Intentionally Alter IAM Permissions (Revoke DynamoDB Write)
1. Open **IAM Console** $\rightarrow$ Click **Roles**.
2. Open role **`LambdaMediaProcessingRole`**.
3. Under the **Permissions** tab, locate **`LambdaMediaPipelinePolicy`** $\rightarrow$ Click **Edit**.
4. In the **JSON** editor, modify the DynamoDB action statement from:
   ```json
   "Action": [
     "dynamodb:PutItem"
   ]
   ```
   To:
   ```json
   "Action": [
     "dynamodb:GetItem"
   ]
   ```
   *(Intentionally strips `PutItem` write authorization from Lambda).*
5. Click **Next** $\rightarrow$ Click **Save changes**.
![Test8](/images/5-Workshop/5.10-Testing/Test8.png)
#### Step 2: Trigger Operational Failure via Web Portal
1. Return to the Web Portal `http://fcaj-media-portal-web-2026.s3-website-ap-southeast-2.amazonaws.com`.
2. Under Section 1 (Upload), select a new file (e.g., `error-test.png`) $\rightarrow$ Click **Upload to S3 Cloud**.
3. S3 accepts the binary upload and triggers the Lambda event consumer.

#### Step 3: Inspect Exception Stack Trace in CloudWatch Logs
1. Open **CloudWatch Console** $\rightarrow$ Go to **Logs** $\rightarrow$ **Log groups**.
2. Open `/aws/lambda/process-media-metadata` $\rightarrow$ Open the most recent Log stream.
3. Observe the captured failure trace:
   ```text
   [ERROR] ClientError: An error occurred (AccessDeniedException) when calling the PutItem operation: User: arn:aws:sts::508266023015:assumed-role/LambdaMediaProcessingRole/process-media-metadata is not authorized to perform: dynamodb:PutItem on resource: arn:aws:dynamodb:ap-southeast-2:508266023015:table/MediaMetadata
   Traceback (most recent call last):
     File "/var/task/lambda_function.py", line 125, in lambda_handler
       table.put_item(Item=...)
   ```

#### Step 4: Monitor CloudWatch Alarm State Transition
1. In CloudWatch sidebar, navigate to **Alarms** $\rightarrow$ **All alarms**.
2. Select **`MediaLambdaFailureAlarm`**.
3. After 1 to 2 minutes, observe the **State** indicator:
   - Status switches from **OK** to a red indicator: **In alarm**.
   - Metric chart indicates `Errors` spiked to `1.0`.

#### Step 5: Verify Automated Incident Email Dispatch
1. Check your personal email inbox.
2. Confirm arrival of an urgent notification email from AWS Notifications:
   ```text
   ALARM: "MediaLambdaFailureAlarm" in Asia Pacific (Sydney)
   ```
3. The email payload states that the `Errors` metric for `process-media-metadata` breached the $\ge 1$ threshold during the 300-second evaluation period.

#### Step 6: Remediate Permissions to Normal Operation
1. Return to IAM Console $\rightarrow$ Role `LambdaMediaProcessingRole` $\rightarrow$ Edit `LambdaMediaPipelinePolicy`.
2. Revert `"dynamodb:GetItem"` back to `"dynamodb:PutItem"`.
3. Click **Save changes**.
4. Within approximately 5 minutes of clean execution, `MediaLambdaFailureAlarm` returns to the green **OK** status.

**Checkpoint:** CloudWatch Alarm successfully entered the alarm state and triggered email alerts, validating 100% of the Fault Injection testing criteria.

---

## 3. Expected Outcomes

- Validated end-to-end serverless data pipeline: Web Portal $\rightarrow$ API Gateway $\rightarrow$ S3 Data $\rightarrow$ Lambda $\rightarrow$ DynamoDB $\rightarrow$ SNS Email.
- Confirmed SigV4 presigned digital signatures for direct client uploads (PUT) and secure downloads (GET).
- Verified Fault Injection response: CloudWatch Logs captured `AccessDeniedException` stack traces, CloudWatch Alarms entered `In alarm`, and emergency alerts were fanned out via Amazon SNS.
- System operational resilience fully confirmed, ready to advance to resource teardown in Section 5.11.