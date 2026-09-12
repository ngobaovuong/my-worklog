---
title : "SNS Notification Setup"
date : 2026-09-25
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

### Objectives

Provision an **Amazon Simple Notification Service (SNS) Topic** utilizing the Publish/Subscribe (Pub/Sub) messaging paradigm, configure an **Email Subscription** for real-time alerts, complete secure cryptographic email confirmation, and retrieve the Topic Amazon Resource Name (ARN) for integration into AWS Lambda and CloudWatch Alarms.

---

## 1. Architectural Concepts & Pub/Sub Messaging

Decoupling compute engines from notification destinations is a core design pillar of resilient cloud architectures:

- **Publisher / Subscriber (Pub/Sub) Architecture:**
  - **Publisher:** Compute components (AWS Lambda) or telemetry monitors (CloudWatch Alarms) push event payloads to an ingestion bus designated as a **Topic**. The publisher remains completely decoupled from endpoint protocols, oblivious to recipient identities, counts, or network locations.
  - **Subscriber:** Downstream targets subscribed to the Topic (in this scenario, the administrator's personal email inbox). Whenever an event message lands on the Topic, SNS fans out the payload instantaneously to all validated subscribers.
- **SNS Topic Classification:**
  - **Standard Topic (Selected):** Offers near-unlimited transactions per second (TPS), delivers best-effort message ordering, ensures single-digit millisecond routing latencies, and natively supports outgoing Email delivery protocols.
  - **FIFO Topic:** Guarantees strict message deduplication and ordering, but restricts delivery to programmatic queues (Amazon SQS FIFO) rather than human-readable email channels.
- **Subscription Lifecycle & Verification:**
  - Newly declared email subscriptions initially reside in a **PendingConfirmation** state.
  - AWS issues an automated verification email containing an authenticated handshake token. The subscription only transitions to **Confirmed** once the recipient acknowledges ownership by following the verification link.

---

## 2. Step-by-Step Instructions

### Step 1: Provision the Standard SNS Topic

1. Sign in to the **AWS Management Console** and ensure the active Region indicates **Asia Pacific (Sydney) ap-southeast-2**.
2. Type `SNS` into the top search bar and select **Simple Notification Service**.
3. In the left navigation pane, select **Topics** $\rightarrow$ Click the orange **Create topic** button.
![AmazonSNS1](/images/5-Workshop/5.5-SNS-Notification/AmazonSNS1.png)
4. Under the **Details** configuration panel:
   - **Type**: Select **Standard**.
   - **Name**: Enter:
     ```text
     MediaProcessingAlerts
     ```
   - **Display name** (acts as the email sender display prefix): Enter:
     ```text
     MediaVault
     ```
5. Leave all settings under **Encryption**, **Access policy**, **Data protection policy**, and **Delivery status logging** at their default values.
6. Scroll to the bottom and click **Create topic**.

```text
Type: Standard
Name: MediaProcessingAlerts
Display name: MediaVault
```

**Checkpoint:** The console displays the `MediaProcessingAlerts` details dashboard. In the **Details** card, copy the **Topic ARN** string for use in the Lambda compute implementation:
```text
arn:aws:sns:ap-southeast-2:<ACCOUNT-ID>:MediaProcessingAlerts
```

---

### Step 2: Configure the Email Subscription

1. On the `MediaProcessingAlerts` details page, scroll down to the tab collection and select **Subscriptions**.
2. Click the orange **Create subscription** button.
3. On the **Create subscription** configuration page:
   - **Topic ARN**: Automatically populated with the ARN of `MediaProcessingAlerts`.
   - **Protocol**: Click the dropdown menu and select **Email**.
   - **Endpoint**: Enter your personal email address (the recipient address for file processing summaries and system error notifications).
4. Retain default options for **Subscription filter policy** and **Redrive policy (dead-letter queue)**.
5. Click **Create subscription**.
![AmazonSNS2](/images/5-Workshop/5.5-SNS-Notification/AmazonSNS2.png)
```text
Protocol: Email
Endpoint: your-personal-email@example.com
```

**Checkpoint:** The console redirects back to the Topic details view. Under the **Subscriptions** tab, the newly registered endpoint appears with its **Status** displayed in red: `PendingConfirmation`.

---

### Step 3: Complete Email Verification

1. Open a new browser tab and navigate to the email inbox configured in Step 2.
2. Check the Inbox or Spam/Junk folders.
3. Locate the incoming email dispatched by **AWS Notifications** (`no-reply@sns.amazonaws.com`) with the subject:
   ```text
   AWS Notification - Subscription Confirmation
   ```
4. Open the email message and click the **Confirm subscription** link.
![AmazonSNS3](/images/5-Workshop/5.5-SNS-Notification/AmazonSNS3.png)
5. The browser opens an official AWS landing page confirming the subscription:
   ```text
   Subscription confirmed!
   You have successfully subscribed to the topic: MediaProcessingAlerts.
   ```
6. Return to the AWS Console browser tab and click the **Refresh** button on the Subscriptions table.
![AmazonSNS4](/images/5-Workshop/5.5-SNS-Notification/AmazonSNS4.png)
**Checkpoint:** The **Status** column for the email address changes from `PendingConfirmation` to a green **Confirmed** label, and an active Subscription ID replaces the pending placeholder.

---

## 3. Expected Outcomes

- Successfully created a Standard SNS Topic named `MediaProcessingAlerts` in the **ap-southeast-2 (Sydney)** Region.
- Successfully attached an Email Subscription and validated ownership, achieving **Confirmed** operational status.
- Retrieved and noted the valid `Topic ARN` for subsequent parameterization in the Python code of the AWS Lambda compute function.
- Push alerting infrastructure staged to handle both informational media ingestion reports and urgent error triggers emitted by CloudWatch Alarms.