---
title : "IAM Security Configuration"
date : 2026-09-25
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

### Objectives

Provision and configure an **IAM Execution Role** for the AWS Lambda function adhering strictly to the Principle of Least Privilege. Identify, prevent, and remediate permission denials caused by **Permissions Boundaries**, ensuring Lambda seamlessly interacts with S3, DynamoDB, SNS, and CloudWatch Logs.

---

## 1. Theoretical Concepts & IAM Architecture

In a serverless event-driven architecture, AWS Lambda acts as the central compute intermediary. By default, a newly created Lambda function has no ambient permissions to access storage tiers or database endpoints within the AWS cloud.

The architecture enforces precise service-level delegations:
- **CloudWatch Logs:** Requires `logs:CreateLogGroup`, `logs:CreateLogStream`, and `logs:PutLogEvents` to record operational traces. This is granted via the AWS Managed Policy: `AWSLambdaBasicExecutionRole`.
- **Amazon S3:** Requires `s3:GetObject` (to read file payloads) and `s3:PutObject` (to issue upload presigned URLs), strictly restricted to the `fcaj-media-source-*` bucket prefix.
- **Amazon DynamoDB:** Requires `dynamodb:PutItem` to write file metadata records into the `MediaMetadata` table.
- **Amazon SNS:** Requires `sns:Publish` to broadcast structured processing summaries to the `MediaProcessingAlerts` Topic.

### Permissions Boundary Troubleshooting Insight:
In specific enterprise or sandbox accounts, roles may inherit an implicit **Permissions Boundary**. Boundaries establish maximum permissible boundaries: even if an identity policy explicitly grants `dynamodb:PutItem`, the action is blocked if omitted from the boundary, resulting in:
`...is not authorized to perform: dynamodb:PutItem ... because no permissions boundary allows the dynamodb:PutItem action`.
Therefore, this procedure explicitly audits and removes any applied Permissions Boundary.

---

## 2. Step-by-Step Instructions

### Step 1: Create the Lambda Execution Role

1. Sign in to the **AWS Management Console** (verify the active Region is **ap-southeast-2 Sydney**).
2. Type `IAM` in the top search bar and open the **IAM** service.
![IAM Page](/images/5-Workshop/5.3-IAM-Setup/IAM1.png)
3. In the left navigation pane, select **Roles** $\rightarrow$ Click the orange **Create role** button.
4. On the **Select trusted entity** page:
   - **Trusted entity type**: Select **AWS service**.
   - **Use case**: Choose **Lambda** from the list of common use cases.
5. Click **Next** at the bottom-right corner.
![IAM Role Page](/images/5-Workshop/5.3-IAM-Setup/IAM2.png)
```text
Trusted entity type: AWS service
Use case: Lambda
```

**Checkpoint:** The console progresses to the **Add permissions** step.

---

### Step 2: Attach Baseline Logging Managed Policy

1. In the **Search policies** field, enter: `AWSLambdaBasicExecutionRole`.
2. Press Enter and check the checkbox adjacent to **AWSLambdaBasicExecutionRole**.
3. Click **Next**.
![Add permissions Page](/images/5-Workshop/5.3-IAM-Setup/IAM4.png)

**Checkpoint:** The console advances to the **Name, review, and create** step.

---

### Step 3: Name Role and Audit Permissions Boundaries

1. In the **Role name** field, input exactly:
   ```text
   LambdaMediaProcessingRole
   ```
2. Under **Description**, add an operational description:
   ```text
   Role for Lambda to process S3 media events, write to DynamoDB, and publish to SNS.
   ```
3. Scroll down to the **Permissions boundary** section:
   - Carefully verify that it indicates: **No boundary set**.
   - *Critical Notice:* If a boundary policy is currently populated, click the button to remove or clear the boundary setting.
4. Scroll to the bottom and click **Create role**.

**Checkpoint:** A success banner confirms `Role LambdaMediaProcessingRole created`. Search for `LambdaMediaProcessingRole` in the role list and click its name to open the detailed settings page.

---

### Step 4: Attach Custom Inline Policy for S3, DynamoDB, and SNS

1. On the `LambdaMediaProcessingRole` details page, ensure you are on the **Permissions** tab.
2. Click **Add permissions** $\rightarrow$ Select **Create inline policy**.
3. In the policy editor interface, click the **JSON** tab located in the top-right header of the editor block.
4. Clear the prefilled template and paste the following policy definition:
![IAM Policy Page](/images/5-Workshop/5.3-IAM-Setup/IAM3.png)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3ObjectAccess",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::fcaj-media-source-*/*"
    },
    {
      "Sid": "DynamoDBAccess",
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-2:*:table/MediaMetadata"
    },
    {
      "Sid": "SNSPublishAccess",
      "Effect": "Allow",
      "Action": [
        "sns:Publish"
      ],
      "Resource": "arn:aws:sns:ap-southeast-2:*:MediaProcessingAlerts"
    }
  ]
}
```

5. Click **Next** in the bottom-right corner.
6. On the **Review and create** step:
   - **Policy name**: Enter:
     ```text
     LambdaMediaPipelinePolicy
     ```
7. Click **Create policy**.

![IAM Policy Page2](/images/5-Workshop/5.3-IAM-Setup/IAM5.png)

**Checkpoint:** The **Permissions policies** list on `LambdaMediaProcessingRole` contains exactly 2 policies:
- `AWSLambdaBasicExecutionRole` (AWS managed policy)
- `LambdaMediaPipelinePolicy` (Customer inline policy)

---

## 3. Expected Outcomes

- Successfully provisioned IAM Role `LambdaMediaProcessingRole` with a valid trust relationship allowing `lambda.amazonaws.com` execution.
- Configured granular permissions enabling automatic CloudWatch log stream generation, S3 Put/Get object actions, DynamoDB writes, and SNS alert broadcasts.
- Verified that no restrictive Permissions Boundary exists to block runtime Lambda actions.
- Foundation access control staged before creating S3 data buckets and the DynamoDB metadata store in the next chapter.
![Role](/images/5-Workshop/5.3-IAM-Setup/IAM6.png)