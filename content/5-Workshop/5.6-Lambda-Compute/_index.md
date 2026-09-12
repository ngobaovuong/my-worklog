---
title : "Compute Logic with AWS Lambda"
date : 2026-09-25
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

### Objectives

Build, configure, and deploy the core compute logic in **AWS Lambda** (Python 3.12) to serve as the system processing engine. The function handles dual workflows: provisioning SigV4-signed **S3 Presigned URLs** (PUT/GET) for browser clients via Amazon API Gateway, and automatically parsing file metadata, persisting audit logs into DynamoDB, and publishing structured incident alerts to Amazon SNS upon receiving `s3:ObjectCreated:*` events.

---

## 1. Architectural Concepts & Handler Routing

The Lambda function is authored in Python 3.12 leveraging the **Boto3 SDK**, implementing an event-driven dual-routing architecture:

### 1. Synchronous API Gateway Ingress (Request-Response Flow):
- When users interact with the Web Portal (initiating file uploads or downloads), API Gateway proxies the raw HTTP payload as a JSON event into Lambda.
- **CORS Preflight Handling (`OPTIONS`):** Returns immediate HTTP status 200 with headers `Access-Control-Allow-Origin: *` to pass browser preflight security handshakes.
- **Upload Presigned URL Generation (`PUT`):** Invokes `s3_client.generate_presigned_url(ClientMethod='put_object', ...)` configured with a 300-second (5-minute) lifetime. The client uses this signed URL to stream binary data directly to S3 without IAM credentials.
- **Download Presigned URL Generation (`GET`):** Invokes `generate_presigned_url(ClientMethod='get_object', ...)` to synthesize temporary direct retrieval links.
- **SigV4 Regional Endpoint Insight:** To eliminate browser HTTP 307 temporary redirects and corresponding CORS cross-origin blocks caused by default S3 global endpoints, the Boto3 client explicitly locks down regional settings:
  ```python
  config = Config(signature_version='s3v4')
  endpoint_url = '[https://s3.ap-southeast-2.amazonaws.com](https://s3.ap-southeast-2.amazonaws.com)'
  ```

### 2. Asynchronous S3 Ingress (Event Consumer Flow):
- Following successful binary uploads into the S3 Data Bucket, S3 triggers the Lambda function asynchronously with an event containing the `Records` array.
- **URL Decoding (`urllib.parse.unquote_plus`):** S3 replaces whitespace with `+` or `%20`. Decoding ensures the exact filename is captured.
- **DynamoDB Audit Persistence:** Records metadata attributes into `MediaMetadata`, including `FileId`, `UploadTime`, `Bucket`, `SizeBytes`, `Format`, and `Status`.
- **SNS Alert Fan-out:** Formats transaction digests into human-readable ASCII tables detailing object sizes in Bytes and KB, then publishes directly to the `MediaProcessingAlerts` SNS Topic.

---

## 2. Step-by-Step Instructions

### Step 1: Provision the AWS Lambda Function

1. Sign in to the **AWS Management Console** and confirm the active Region is **Asia Pacific (Sydney) ap-southeast-2**.
2. Type `Lambda` in the top search bar and open the **Lambda** service.
3. On the Functions dashboard, click the orange **Create function** button.
![Lambda](/images/5-Workshop/5.6-Lambda-Compute/Lambda.png)
4. Select **Author from scratch**:
   - **Function name**: Enter:
     ```text
     process-media-metadata
     ```
   - **Runtime**: Select **Python 3.12**.
   - **Architecture**: Select **x86_64**.
5. Expand the **Change default execution role** section:
   - Select **Use an existing role**.
   - **Existing role**: Select `LambdaMediaProcessingRole` configured in Section 5.3.
6. Click the orange **Create function** button in the bottom-right corner.
![Lambda2](/images/5-Workshop/5.6-Lambda-Compute/Lambda2.png)
```text
Function name: process-media-metadata
Runtime: Python 3.12
Architecture: x86_64
Execution role: Use an existing role (LambdaMediaProcessingRole)
```

**Checkpoint:** The console navigates to the `process-media-metadata` detail view with a green banner confirming function creation.
![Lambda3](/images/5-Workshop/5.6-Lambda-Compute/Lambda3.png)
---

### Step 2: Implement Compute Logic & Deploy

1. In the function details view, scroll down to the **Code source** editor.
2. Double-click `lambda_function.py` in the left file tree.
3. Erase all template code and paste the following complete script:
![Lambda4](/images/5-Workshop/5.6-Lambda-Compute/Lambda4.png)
![Lambda5](/images/5-Workshop/5.6-Lambda-Compute/Lambda5.png)
![Lambda6](/images/5-Workshop/5.6-Lambda-Compute/Lambda6.png)
```python
import json
import urllib.parse
import boto3
from datetime import datetime
from botocore.config import Config

# Explicit SigV4 Boto3 Client locked to the Sydney Regional Endpoint
s3_client = boto3.client(
    's3',
    region_name='ap-southeast-2',
    endpoint_url='[https://s3.ap-southeast-2.amazonaws.com](https://s3.ap-southeast-2.amazonaws.com)',
    config=Config(signature_version='s3v4')
)
dynamodb = boto3.resource('dynamodb', region_name='ap-southeast-2')
sns_client = boto3.client('sns', region_name='ap-southeast-2')

# Target infrastructure constants
BUCKET_NAME = "fcaj-media-source-demo-2026"
TABLE_NAME = "MediaMetadata"
# REPLACE THE STRING BELOW WITH YOUR ACTUAL SNS TOPIC ARN
SNS_TOPIC_ARN = "arn:aws:sns:ap-southeast-2:508266023015:MediaProcessingAlerts"

def lambda_handler(event, context):
    headers = {
        "Access-Control-Allow-Origin": "*",
        "Access-Control-Allow-Headers": "Content-Type",
        "Access-Control-Allow-Methods": "OPTIONS,POST,GET"
    }

    # =========================================================================
    # 1. API GATEWAY ROUTE: Generate Presigned URLs
    # =========================================================================
    if 'httpMethod' in event or 'requestContext' in event:
        http_method = event.get('httpMethod', '')
        
        # Handle CORS Preflight handshake
        if http_method == 'OPTIONS':
            return {
                'statusCode': 200,
                'headers': headers,
                'body': ''
            }
            
        body = json.loads(event.get('body', '{}')) if event.get('body') else {}
        action = body.get('action')
        file_name = body.get('fileName')

        if not file_name:
            return {
                'statusCode': 400,
                'headers': headers,
                'body': json.dumps({'error': 'fileName is required in request payload'})
            }

        # Issue temporary upload URL (PUT)
        if action == 'get_upload_url':
            presigned_url = s3_client.generate_presigned_url(
                ClientMethod='put_object',
                Params={
                    'Bucket': BUCKET_NAME,
                    'Key': file_name
                },
                ExpiresIn=300
            )
            return {
                'statusCode': 200,
                'headers': headers,
                'body': json.dumps({
                    'uploadUrl': presigned_url,
                    'fileName': file_name
                })
            }

        # Issue temporary download URL (GET)
        elif action == 'get_download_url':
            presigned_url = s3_client.generate_presigned_url(
                ClientMethod='get_object',
                Params={
                    'Bucket': BUCKET_NAME,
                    'Key': file_name
                },
                ExpiresIn=300
            )
            return {
                'statusCode': 200,
                'headers': headers,
                'body': json.dumps({
                    'downloadUrl': presigned_url
                })
            }

    # =========================================================================
    # 2. S3 ASYNCHRONOUS EVENT ROUTE: Ingest Metadata & Alert
    # =========================================================================
    if 'Records' in event and 's3' in event['Records'][0]:
        table = dynamodb.Table(TABLE_NAME)
        
        for record in event['Records']:
            bucket = record['s3']['bucket']['name']
            # Decode URL safe characters (spaces and special symbols)
            key = urllib.parse.unquote_plus(record['s3']['object']['key'])
            size_bytes = record['s3']['object']['size']
            upload_time = datetime.utcnow().strftime('%Y-%m-%d %H:%M:%S UTC')
            file_extension = key.split('.')[-1].upper() if '.' in key else 'UNKNOWN'

            # Persist audit record to DynamoDB
            table.put_item(
                Item={
                    'FileId': key,
                    'UploadTime': upload_time,
                    'Bucket': bucket,
                    'SizeBytes': size_bytes,
                    'Format': file_extension,
                    'Status': 'ACTIVE'
                }
            )

            # Format structured summary for Amazon SNS
            size_kb = round(size_bytes / 1024, 2)
            detailed_msg = (
                "===========================================\n"
                "       AWS MEDIA VAULT - NEW UPLOAD        \n"
                "===========================================\n"
                f"• File Name      : {key}\n"
                f"• Extension      : {file_extension}\n"
                f"• Size           : {size_bytes} Bytes (~{size_kb} KB)\n"
                f"• Target Bucket  : {bucket}\n"
                f"• Uploaded At    : {upload_time}\n"
                f"• Database Sync  : DynamoDB [MediaMetadata] - SUCCESS\n"
                "===========================================\n"
                "Status: Encrypted and ready for secure retrieval."
            )

            sns_client.publish(
                TopicArn=SNS_TOPIC_ARN,
                Subject=f"[AWS Media Vault] File {key} Processed Successfully",
                Message=detailed_msg
            )

        return {
            'statusCode': 200,
            'body': json.dumps('S3 Event Handled and Processed Successfully')
        }

    return {
        'statusCode': 400,
        'headers': headers,
        'body': json.dumps({'error': 'Invalid event trigger or unhandled route'})
    }
```

4. Confirm that `SNS_TOPIC_ARN` and `BUCKET_NAME` match your deployed AWS assets.
5. Click the blue **Deploy** button on the code editor toolbar.

**Checkpoint:** The banner displays `Successfully deployed changes`.

---

### Step 3: Attach the S3 Trigger

1. In the **Function overview** panel at the top, click **+ Add trigger**.
2. Under **Select a source**: Search for and select **S3**.
3. Configure the trigger properties:
   - **Bucket**: Choose `fcaj-media-source-demo-2026`.
   - **Event types**: Select **All object create events** (`s3:ObjectCreated:*`).
   - **Prefix**: Leave empty.
   - **Suffix**: Leave empty (to capture all image and video extensions).
4. Acknowledge the recursive invocation notification:
   - *"I understand that using the same S3 bucket for input and output is not recommended and can cause recursive invocations."*
5. Click **Add**.

```text
Trigger Source: S3
Bucket: fcaj-media-source-demo-2026
Event type: All object create events
Recursive invocation acknowledgment: Checked
```

**Checkpoint:** In the **Function overview** diagram, the **S3** trigger block is visibly attached to the left input of the Lambda function.

---

### Step 4: Validate via Console Mock Unit Testing

To verify execution logic and IAM permissions in isolation before assembling API Gateway:

1. Switch to the **Test** tab adjacent to the Code tab.
2. Under **Test event action**: Select **Create new event**.
3. **Event name**: Enter `TestS3UploadEvent`.
4. In the **Template** dropdown: Select the **s3-put** template.
5. In the JSON event editor, adjust these two properties:
   - Change `"name": "example-bucket"` $\rightarrow$ to: `"name": "fcaj-media-source-demo-2026"`
   - Change `"key": "test%2Fkey"` $\rightarrow$ to: `"key": "test-manual-upload.png"`
6. Click **Save** in the top right, then click the orange **Test** button.
7. Inspect the execution output:
   - Green execution output confirms: **Execution result: Succeeded (Status: 200)**.
   - Check DynamoDB table `MediaMetadata`: The record `test-manual-upload.png` is populated.
   - Check email inbox: An automated delivery notification from SNS arrives.

**Checkpoint:** Unit testing succeeds with Status 200, validating that Lambda permissions are operational and ready for API Gateway integration in Section 5.7.

---

## 3. Expected Outcomes

- Successfully created and configured `process-media-metadata` Lambda function on Python 3.12 in Sydney `ap-southeast-2`.
- Deployed dual-purpose logic for signing S3 Presigned URLs and processing asynchronous S3 events.
- Connected S3 Bucket `fcaj-media-source-demo-2026` as an automated event trigger.
- Verified end-to-end execution through console unit tests, confirming IAM permissions and preparing the system for Amazon API Gateway deployment in Section 5.7.