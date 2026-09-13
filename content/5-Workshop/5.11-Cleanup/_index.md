---
title : "Resource Cleanup"
date : 2026-09-13
weight : 11
chapter : false
pre : " <b> 5.11. </b> "
---

### Objectives

Execute an orderly, comprehensive teardown of all cloud resources provisioned throughout the workshop. Proper resource disposal enforces the **Cost Optimization** pillar of the AWS Well-Architected Framework, eliminating residual billing risks following the conclusion of your evaluation or the expiration of the AWS Free Tier.

---

## 1. Theoretical Concepts & Safe Disposal Principles

Cloud assets, even when dormant, incur ongoing metering charges if left provisioned within an AWS account:
- **S3 & DynamoDB Storage Overhead:** Storage costs accumulate continuously based on total object sizes and database disk utilization.
- **Resource Dependency Constraints:**
  - Amazon S3 strictly rejects deletion requests for non-empty buckets. Buckets must be emptied using the **Empty Bucket** workflow prior to deletion.
  - Event triggers bound to S3 buckets detach automatically when associated compute functions or source buckets are removed.
- **Recommended Teardown Sequence:**
  1. Edge ingress and presentation layers (S3 Web Hosting, API Gateway).
  2. Compute layer (AWS Lambda).
  3. Storage and database layers (S3 Data Bucket, DynamoDB Table).
  4. Observability and alerting assets (CloudWatch Alarms, CloudWatch Logs, SNS Topics).
  5. Security credentials and delegations (IAM Roles).

---

## 2. Step-by-Step Instructions

### Step 1: Empty and Delete Amazon S3 Buckets

The architecture contains 2 buckets: the private data store (`fcaj-media-source-demo-2026`) and the public web portal (`fcaj-media-portal-web-2026`).

#### 1. Delete S3 Data Bucket (`fcaj-media-source-demo-2026`):
1. Sign in to the **AWS Management Console** $\rightarrow$ Confirm Region is **Asia Pacific (Sydney) ap-southeast-2**.
2. Open the **Amazon S3** service.
3. In the Buckets table, select the checkbox next to **`fcaj-media-source-demo-2026`**.
4. Click the **Empty** button on the top toolbar.
5. In the confirmation input box, type:
   ```text
   permanently delete
   ```
6. Click the orange **Empty** button in the bottom-right corner to purge all stored images/videos.
7. Click **Exit** to return to the Buckets dashboard.
8. Re-select the checkbox for **`fcaj-media-source-demo-2026`** $\rightarrow$ Click **Delete**.
9. In the confirmation dialog, enter the exact bucket name:
   ```text
   fcaj-media-source-demo-2026
   ```
10. Click **Delete bucket**.

#### 2. Delete S3 Web Hosting Bucket (`fcaj-media-portal-web-2026`):
1. Select the checkbox next to **`fcaj-media-portal-web-2026`**.
2. Click **Empty** $\rightarrow$ Enter `permanently delete` $\rightarrow$ Click **Empty**.
3. Click **Exit**.
4. Re-select **`fcaj-media-portal-web-2026`** $\rightarrow$ Click **Delete**.
5. Enter the bucket name to confirm:
   ```text
   fcaj-media-portal-web-2026
   ```
6. Click **Delete bucket**.

**Checkpoint:** Both buckets are completely removed from the Amazon S3 Buckets inventory.

---

### Step 2: Delete Amazon API Gateway REST API

1. Type `API Gateway` in the top search bar and select **API Gateway**.
2. In the APIs dashboard, locate **`MediaPortalAPI`**.
3. Check the box adjacent to the API name (or click the three-dot icon / **Actions** menu).
4. Select **Delete**.
5. In the confirmation dialog, click **Delete**.

**Checkpoint:** REST API `MediaPortalAPI` is permanently removed, invalidating all Invoke URL endpoints.

---

### Step 3: Delete AWS Lambda Function

1. Type `Lambda` in the search bar and select **Lambda**.
2. In the left navigation pane, select **Functions**.
3. Select the checkbox next to: **`process-media-metadata`**.
4. Click the **Actions** dropdown menu $\rightarrow$ Select **Delete**.
5. In the confirmation dialog, type:
   ```text
   delete
   ```
6. Click **Delete**.

**Checkpoint:** The function `process-media-metadata` is deleted from the Lambda console.

---

### Step 4: Delete Amazon DynamoDB Table

1. Type `DynamoDB` in the search bar and open the **DynamoDB** console.
2. In the left navigation pane, select **Tables**.
3. Select the checkbox for table: **`MediaMetadata`**.
4. Click the **Delete** button in the top action bar.
5. In the modal confirmation prompt:
   - Type:
     ```text
     confirm
     ```
   - Uncheck automated CloudWatch alarm deletion options if prompted.
6. Click **Delete table**.

**Checkpoint:** Table `MediaMetadata` enters *Deleting* status and is purged within seconds.

---

### Step 5: Clean Up Amazon CloudWatch (Alarm & Log Group)

#### 1. Delete CloudWatch Alarm:
1. Open the **Amazon CloudWatch** console.
2. In the left sidebar, navigate to **Alarms** $\rightarrow$ **All alarms**.
3. Check the box for: **`MediaLambdaFailureAlarm`**.
4. Click **Actions** $\rightarrow$ Select **Delete**.
5. Confirm deletion by clicking **Delete**.

#### 2. Delete CloudWatch Log Group:
1. In the CloudWatch sidebar, navigate to **Logs** $\rightarrow$ **Log groups**.
2. In the search filter, enter:
   ```text
   /aws/lambda/process-media-metadata
   ```
3. Select the checkbox for `/aws/lambda/process-media-metadata`.
4. Click **Actions** $\rightarrow$ Select **Delete log group(s)**.
5. Click **Delete** in the confirmation modal.

**Checkpoint:** The failure alarm and execution log streams are purged, freeing CloudWatch log storage space.

---

### Step 6: Delete Amazon SNS (Topic & Subscriptions)

1. Open the **Amazon SNS** console.
2. In the left navigation pane, select **Topics**.
3. Select the checkbox for topic: **`MediaProcessingAlerts`**.
4. Click the **Delete** button in the top-right corner.
5. In the confirmation prompt, type:
   ```text
   delete me
   ```
6. Click **Delete**.
7. Navigate to **Subscriptions** in the left navigation pane:
   - If an orphaned subscription targeting your personal email remains, select it $\rightarrow$ Click **Delete** $\rightarrow$ Confirm deletion.

**Checkpoint:** Topic and subscription associations are purged from Amazon SNS.

---

### Step 7: Delete AWS IAM Execution Role

1. Open the **IAM** console.
2. In the left navigation pane, select **Roles**.
3. In the search bar, enter: `LambdaMediaProcessingRole`.
4. Select the checkbox next to **`LambdaMediaProcessingRole`**.
5. Click the **Delete** button in the top-right corner.
6. In the confirmation dialog, enter the exact role name:
   ```text
   LambdaMediaProcessingRole
   ```
7. Click **Delete**.

**Checkpoint:** Role `LambdaMediaProcessingRole` along with its attached inline policies is purged from the AWS account.

---

## 3. Expected Outcomes

- Complete disposal of all 7 AWS architecture components (S3, API Gateway, Lambda, DynamoDB, CloudWatch, SNS, IAM) in Sydney `ap-southeast-2`.
- Zero active background compute triggers, orphaned object storage, or provisioned database capacity.
- The AWS account is returned to a clean baseline state, guaranteeing post-workshop operational expenditure remains at **$0.00 USD**.