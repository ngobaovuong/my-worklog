---
title : "Monitoring & Alarms with CloudWatch"
date : 2026-09-25
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

### Objectives

Establish comprehensive observability adhering to the **Operational Excellence** pillar of the AWS Well-Architected Framework. Leverage **Amazon CloudWatch Logs** to analyze execution traces from the serverless compute layer, monitor runtime telemetry via **CloudWatch Metrics**, and configure proactive **CloudWatch Alarms** linked with Amazon SNS to trigger instant emergency email dispatches upon system failures.

---

## 1. Theoretical Concepts & Monitoring Topology

In decoupled serverless architectures, the absence of physical server access requires robust log aggregation and telemetry metrics:

- **Amazon CloudWatch Logs & Log Group Mechanics:**
  - Each Lambda invocation automatically redirects standard output streams (`stdout`, `stderr`, and internal `print()` logs) directly into CloudWatch.
  - All logs are aggregated into a standardized **Log Group**:
    ```text
    /aws/lambda/process-media-metadata
    ```
  - Within this Log Group, distinct execution container instances dynamically provision their own discrete **Log Streams**.
  - Standard execution log events comprise three distinct stages:
    - `START RequestId`: Invocation initialization.
    - Application Logs: Custom emitted diagnostics (e.g., URL decoding traces, database payloads).
    - `REPORT RequestId`: Resource telemetry containing `Duration` (actual execution time), `Billed Duration` (metered execution increments), `Memory Size` (allocated RAM), and `Max Memory Used` (peak runtime RAM consumption).
  - *Troubleshooting Empty Log Groups (`Log groups: 0`):* If Lambda invocations execute but CloudWatch indicates "There are no log groups", the execution role lacks essential logging permissions. Attaching the managed policy `AWSLambdaBasicExecutionRole` (providing `logs:CreateLogGroup`, `logs:CreateLogStream`, and `logs:PutLogEvents`) is mandatory to enable log group auto-creation.

- **Amazon CloudWatch Metrics & Failure Alarms:**
  - AWS Lambda streams runtime telemetry to CloudWatch: `Invocations`, `Duration`, `Errors` (unhandled exceptions or fatal runtimes), and `Throttles`.
  - **Proactive CloudWatch Alarm:** Tracks the `Errors` metric on `process-media-metadata`. If errors reach $\ge 1$ within an evaluation window (5-minute Period), the alarm state changes from **OK** to **In alarm**, firing an automated notification payload into the `MediaProcessingAlerts` SNS Topic to notify administrators immediately.

---

## 2. Step-by-Step Instructions

### Step 1: Inspect and Verify CloudWatch Logs

1. Sign in to the **AWS Management Console** (verify the active Region is **Asia Pacific (Sydney) ap-southeast-2**).
2. Type `CloudWatch` in the top search bar and open the **CloudWatch** service.
![CloudWatch1](/images/5-Workshop/5.9-Monitoring/CloudWatch1.png)
3. In the left navigation pane, expand **Logs** $\rightarrow$ Select **Log groups**.
4. In the filter box, enter:
   ```text
   /aws/lambda/process-media-metadata
   ```
5. Click directly on the matching Log group name.
6. Scroll down to the **Log streams** panel:
   - Click on the top stream entry (displaying the most recent *Last event time*).
7. Inspect the structured log events:
   - Header marker: `START RequestId: ... Version: $LATEST`
   - Application execution logs: Emitted debug statements from `lambda_handler`.
   - Completion marker: `END RequestId: ...`
   - Performance report: `REPORT RequestId: ... Duration: 398.17 ms Billed Duration: 944 ms Memory Size: 128 MB Max Memory Used: 100 MB`

```text
Log Group: /aws/lambda/process-media-metadata
Log Status: Streaming operational logs successfully
```

**Checkpoint:** The console displays live Lambda execution streams, confirming logging permissions are active.

---

### Step 2: Create a CloudWatch Failure Metric Alarm

1. In the left navigation pane, expand **Alarms** $\rightarrow$ Select **All alarms**.
2. Click the orange **Create alarm** button in the top-right corner.
3. Under **Specify metric and conditions**, click the **Select metric** button.
![CloudWatch5Alarm](/images/5-Workshop/5.9-Monitoring/CloudWatch5Alarm.png)
4. On the metric selection modal:
   - Under the **Browse** tab, select the **Lambda** service block.
   - Choose the **By Function Name** category.
   - In the search filter, enter: `process-media-metadata`.
   - Locate the row matching:
     - **Function Name**: `process-media-metadata`
     - **Metric Name**: `Errors`
   - Select the checkbox adjacent to this metric.
5. Click the orange **Select metric** button in the bottom-right corner.

```text
Service: Lambda
Category: By Function Name
Function Name: process-media-metadata
Metric Name: Errors
```
![CloudWatch6Alarm](/images/5-Workshop/5.9-Monitoring/CloudWatch6Alarm.png)
![CloudWatch7Alarm](/images/5-Workshop/5.9-Monitoring/CloudWatch7Alarm.png)
![CloudWatch8Alarm](/images/5-Workshop/5.9-Monitoring/CloudWatch8Alarm.png)
**Checkpoint:** The setup wizard displays the configuration view showing the metric chart for Lambda `Errors`.

---

### Step 3: Configure Alarm Threshold Conditions

1. Under the **Metric** section:
   - **Statistic**: Retain **Sum**.
   - **Period**: Select **5 minutes**.
2. Scroll down to the **Conditions** section:
   - **Threshold type**: Select **Static**.
   - **Whenever Errors is...**: Select **Greater/Equal (>= threshold)**.
   - **than...**: Enter:
     ```text
     1
     ```
     *(Triggers an incident whenever 1 or more errors occur within a 5-minute period).*
3. Under **Additional configuration**:
   - **Datapoints to alarm**: Keep `1 out of 1`.
   - **Missing data treatment**: Choose **Treat missing data as good (not breaching threshold)**.
4. Click **Next** in the bottom-right corner.

---

### Step 4: Configure Urgent Notification via Amazon SNS

1. On the **Configure actions** page:
2. Under **Alarm state trigger**: Select **In alarm**.
3. Under **Send a notification to the following SNS topic**:
   - Select **Select an existing SNS topic**.
   - Under **Send a notification to...**: Select your topic created in Section 5.5:
     ```text
     MediaProcessingAlerts
     ```
   - Confirm that your verified email address displays beneath the topic dropdown.
4. Click **Next** in the bottom-right corner.

```text
Alarm state trigger: In alarm
Notification action: Send a notification to an existing SNS topic
SNS Topic: MediaProcessingAlerts
```

---

### Step 5: Name and Finalize Alarm Creation

1. On the **Add name and description** page:
   - **Alarm name**: Enter:
     ```text
     MediaLambdaFailureAlarm
     ```
   - **Alarm description**: Enter:
     ```text
     Automated alarm triggered when the process-media-metadata Lambda function encounters runtime errors
     ```
2. Click **Next**.
3. On the **Preview and create** page:
   - Review the metric visualization, threshold conditions (`Errors >= 1 within 5 minutes`), and SNS action routing.
4. Scroll to the bottom and click the orange **Create alarm** button.

**Checkpoint:** The alarms dashboard displays `MediaLambdaFailureAlarm`.
- Status initially shows **Insufficient data**.
- Within a few minutes of zero errors, the status transitions to a healthy green **OK**.

---

## 3. Expected Outcomes

- Log Group `/aws/lambda/process-media-metadata` successfully created and ingesting real-time Lambda execution logs.
- CloudWatch Alarm `MediaLambdaFailureAlarm` active, continuously evaluating the `Errors` metric against the $\ge 1$ threshold.
- Alert notification action successfully routed to Amazon SNS Topic `MediaProcessingAlerts`, enabling immediate incident emails upon runtime failures.
- Monitoring and alerting infrastructure staged for comprehensive end-to-end testing and fault-injection verification in Section 5.10.