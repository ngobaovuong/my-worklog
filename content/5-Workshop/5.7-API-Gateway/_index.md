---
title : "API Gateway Integration"
date : 2026-09-25
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

### Objectives

Provision, configure, and deploy **Amazon API Gateway** (REST API) to serve as a secure entry-point proxy connecting the client-side Web Portal with AWS Lambda. Implement multi-tier **CORS** (Cross-Origin Resource Sharing) handshakes and configure **Lambda Proxy Integration** to handle asynchronous S3 Presigned URL synthesis requests.

---

## 1. Architectural Concepts & Ingress Design

In modern cloud-native architectures, browser clients must never interface directly with AWS compute primitives via embedded SDKs (which inevitably expose AWS IAM credentials in client scripts). Instead, **Amazon API Gateway** operates as an enterprise-grade reverse proxy:

- **Regional REST API Topology:** Deployed in the identical AWS Region as S3 and Lambda (**Sydney `ap-southeast-2`**) to minimize cross-region latency and optimize connection handshakes.
- **Lambda Proxy Integration Mechanics:**
  - Forwards the entire raw HTTP request (headers, query parameters, method, JSON body) directly into the Lambda execution context as an `event` parameter.
  - Eliminates the maintenance overhead of Velocity Mapping Templates (VTL), allowing the Python backend to manage routing and status codes natively.
- **Root Cause of `Failed to fetch` & CORS Preflight Mechanics:**
  - Modern browsers sending cross-origin requests containing custom headers (`Content-Type: application/json`) from origins other than the API domain (e.g., S3 Static Hosting or local `file:///` protocols) automatically execute a preflight security probe via the HTTP `OPTIONS` method.
  - If API Gateway fails to respond to `OPTIONS` with standard access headers:
    ```text
    Access-Control-Allow-Origin: *
    Access-Control-Allow-Headers: Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token
    Access-Control-Allow-Methods: OPTIONS,POST,GET
    ```
    The client runtime immediately terminates the transmission, printing `Failed to fetch` into the browser console before the request ever reaches Lambda.
  - *Critical Rule:* Any architectural change to resources, methods, or CORS policies requires an explicit **API Deployment** to the target stage (`prod`) before updates become active on public endpoints.

---

## 2. Step-by-Step Instructions

### Step 1: Provision Regional REST API

1. Sign in to the **AWS Management Console** (verify the active Region is **Asia Pacific (Sydney) ap-southeast-2**).
2. Type `API Gateway` in the top search bar and open the **API Gateway** service.
![APIGateway](/images/5-Workshop/5.7-API-Gateway/APIGateway.png)
3. On the API types landing page, locate the **REST API** card (ensure you select standard *REST API*, **do not** select *REST API Private*) $\rightarrow$ Click **Build**.
4. Configure the **Create REST API** specifications:
   - **Choose the protocol**: Select **REST**.
   - **Create new API**: Select **New API**.
   - **API name**: Enter:
     ```text
     MediaPortalAPI
     ```
   - **Description**: Enter:
     ```text
     API Gateway for AWS Media Vault to issue S3 Presigned URLs
     ```
   - **Endpoint Type**: Select **Regional** (co-located in Sydney with Lambda).
5. Click the orange **Create API** button.
![APIGateway2](/images/5-Workshop/5.7-API-Gateway/APIGateway2.png)
```text
Protocol: REST
Create new API: New API
API name: MediaPortalAPI
Endpoint Type: Regional
```

**Checkpoint:** The console displays the resource tree for `MediaPortalAPI`, showing only the root path `/`.
![APIGateway3](/images/5-Workshop/5.7-API-Gateway/APIGateway3.png)
---

### Step 2: Create the `/media` Resource

1. Click to highlight the root slash `/` in the left resource tree.
2. Click the **Create resource** button in the top action bar.
3. On the **Resource details** page:
   - **Resource path**: Defaults to `/`.
   - **Resource name**: Enter `media`.
   - **Resource path after input**: Automatically populates `/media`.
   - **CORS (Cross-Origin Resource Sharing)**: **Check** this checkbox.
4. Click **Create resource**.
![APIGateway4](/images/5-Workshop/5.7-API-Gateway/APIGateway4.png)
**Checkpoint:** The resource hierarchy displays the child resource `/media` directly under `/`.

---

### Step 3: Configure `ANY` Method with Lambda Proxy Integration

1. Highlight the newly created `/media` resource.
2. Click the **Create method** button in the action bar.
3. Under **Method details**:
   - **Method type**: Select **ANY** from the dropdown menu (catches all HTTP verbs including GET, POST, and OPTIONS).
   - **Integration type**: Select **Lambda function**.
   - **Lambda proxy integration**: **Toggle On** the switch *(Mandatory: enables Lambda to parse `event['body']` and `event['httpMethod']`)*.
   - **Lambda Region**: Confirm **ap-southeast-2** (Sydney).
   - **Lambda function**: Search for and select:
     ```text
     process-media-metadata
     ```
4. Click the orange **Create method** button.
5. If prompted with a dialog to grant invocation permissions (`Add Permission to Lambda Function`), click **OK**.

```text
Method type: ANY
Integration type: Lambda function
Lambda proxy integration: Checked (Enabled)
Lambda function: process-media-metadata
```

**Checkpoint:** Under `/media`, the `ANY` method is active. Selecting `ANY` illustrates the integration pipeline: `Client` $\rightarrow$ `Method Request` $\rightarrow$ `Integration Request (Lambda Proxy)` $\rightarrow$ `process-media-metadata`.

---

### Step 4: Enable and Standardize CORS for `/media`

To ensure browsers do not block POST requests and preflight OPTIONS probes:

1. Select the `/media` resource in the tree.
2. Click **Enable CORS** in the top action bar.
3. Configure the CORS specifications:
   - **Gateway responses**: Check **Default 4XX** and **Default 5XX** (ensures Gateway-generated errors also carry CORS headers).
   - **Methods**: Ensure all exposed methods are selected (**ANY** and **OPTIONS**).
   - **Access-Control-Allow-Origin**: Keep `'*'`.
   - **Access-Control-Allow-Headers**: Keep default `'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'`.
4. Click **Save** in the bottom-right corner.

**Checkpoint:** Under `/media`, an isolated `OPTIONS` method is generated with a Mock 200 response configured with the necessary CORS access headers.

---

### Step 5: Deploy API to the `prod` Stage

All API changes remain staged internally until deployed to an environment:

1. In the top action bar, click the orange **Deploy API** button.
2. In the **Deploy API** modal dialog:
   - **Stage**: Select **\*New Stage\***.
   - **Stage name**: Enter:
     ```text
     prod
     ```
   - **Deployment description**: Enter:
     ```text
     Production deployment for AWS Media Vault
     ```
3. Click **Deploy**.
![APIGateway5](/images/5-Workshop/5.7-API-Gateway/APIGateway5.png)
**Checkpoint:** The console redirects to **Stages** $\rightarrow$ `prod`.

---

### Step 6: Acquire Invoke URL & Validate Endpoint

1. In the `prod` Stage navigation tree, expand `prod` $\rightarrow$ Select the `/media` resource.
2. Locate the blue **Invoke URL** banner at the top of the pane. The complete URI follows this schema:
   ```text
   https://<api-id>[.execute-api.ap-southeast-2.amazonaws.com/prod/media](https://.execute-api.ap-southeast-2.amazonaws.com/prod/media)
   ```
   *(e.g., `https://a1b2c3d4e5.execute-api.ap-southeast-2.amazonaws.com/prod/media`)*.
3. **Copy this complete URL** to your scratchpad. This string represents the `API_ENDPOINT` constant required by the frontend application in Section 5.8.
![APIGateway6](/images/5-Workshop/5.7-API-Gateway/APIGateway6.png)
#### CLI Smoke Test (cURL Verification):
Open a terminal on your local workstation to verify end-to-end connectivity between API Gateway and AWS Lambda:

```bash
curl -X POST https://<api-id>[.execute-api.ap-southeast-2.amazonaws.com/prod/media](https://.execute-api.ap-southeast-2.amazonaws.com/prod/media) \
  -H "Content-Type: application/json" \
  -d '{"action": "get_upload_url", "fileName": "ping-test.png"}'
```

**Expected JSON Response:**
```json
{"uploadUrl": "[https://fcaj-media-source-demo-2026.s3.ap-southeast-2.amazonaws.com/ping-test.png?X-Amz-Algorithm=AWS4-HMAC-SHA256](https://fcaj-media-source-demo-2026.s3.ap-southeast-2.amazonaws.com/ping-test.png?X-Amz-Algorithm=AWS4-HMAC-SHA256)...", "fileName": "ping-test.png"}
```

**Checkpoint:** The cURL command outputs a valid JSON payload containing a signed SigV4 `uploadUrl`, confirming seamless integration between API Gateway and AWS Lambda.

---

## 3. Expected Outcomes

- Provisioned Regional REST API `MediaPortalAPI` within Sydney `ap-southeast-2`.
- Successfully linked `/media` to Lambda function `process-media-metadata` via `ANY` method using Lambda Proxy Integration.
- Configured and deployed CORS preflight mechanisms on `/media`, eliminating `Failed to fetch` connection barriers.
- Deployed API to the `prod` stage and generated an active Invoke URL, staged for integration into the Web Portal in Section 5.8.