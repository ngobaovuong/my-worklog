---
title : "Static Web Hosting on S3"
date : 2026-09-13
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

### Objectives

Provision and configure a dedicated **Amazon S3 Bucket** to serve as a serverless static website host using **S3 Static Website Hosting**. Implement a **Public Read Bucket Policy**, deploy the full client-side interface (`index.html`), and connect the frontend to Amazon API Gateway and S3 Presigned URLs for direct uploads and secure downloads.

---

## 1. Architectural Concepts & Security Decoupling

Following the **Security** and **Cost Optimization** pillars of the AWS Well-Architected Framework:

- **Security Tier Decoupling:**
  - The static presentation asset (`index.html`) is strictly isolated from the private media storage bucket (`fcaj-media-source-demo-2026`).
  - The data bucket enforces **100% Block All Public Access** to protect user assets.
  - The web bucket (`fcaj-media-portal-web-2026`) is scoped for public static file distribution (`s3:GetObject`), acting as a serverless web server.
- **S3 Static Website Hosting Mechanics:**
  - S3 assigns an HTTP website endpoint: `http://<bucket-name>.s3-website-<region>.amazonaws.com`.
  - Incurs zero idle infrastructure costs, charges zero activation fees, and operates well within the AWS Free Tier allowances (5 GB storage, 20,000 GET requests/month).
- **Client Direct Integration Pipeline:**
  1. The user fetches `index.html` from the public S3 Web Hosting Bucket.
  2. Client JavaScript issues an asynchronous handshake to API Gateway (`/media`) to request a signed Presigned URL.
  3. The client uploads the binary payload directly to the private S3 Data Bucket via HTTP `PUT`.
![Web-Hosting](/images/5-Workshop/5.8-Web-Hosting/S3-web-1.png)
---

## 2. Step-by-Step Instructions

### Step 1: Provision the S3 Web Hosting Bucket

1. Sign in to the **AWS Management Console** (verify the active Region is **ap-southeast-2 Sydney**).
2. Type `S3` in the top search bar and open the **S3** service.
3. On the Buckets dashboard, click the orange **Create bucket** button.
4. Configure bucket properties:
   - **Bucket type**: Select **General purpose**.
   - **Bucket name**: Enter a globally unique bucket identifier:
     ```text
     fcaj-media-portal-web-2026
     ```
     *(Note: If taken, append an identifier such as `fcaj-media-portal-web-2026-yourname`).*
   - **AWS Region**: Select **Asia Pacific (Sydney) ap-southeast-2**.
![Web-Hosting2](/images/5-Workshop/5.8-Web-Hosting/S3-web-2.png)
5. Under **Object Ownership**: Select **ACLs disabled (recommended)**.
6. Under **Block Public Access settings for this bucket**:
   - **Uncheck** the **Block all public access** box.
   - In the yellow warning box that appears, **check the acknowledgement checkbox**:
     *"I acknowledge that the current settings might result in this bucket and the objects within becoming public."*
7. Retain remaining defaults $\rightarrow$ Scroll to the bottom and click **Create bucket**.

```text
Bucket name: fcaj-media-portal-web-2026
AWS Region: ap-southeast-2 (Sydney)
Block all public access: Unchecked
Acknowledge public risk: Checked
```

**Checkpoint:** The bucket list displays `fcaj-media-portal-web-2026`, with the *Objects can be public* column displaying the warning status: **Objects can be public**.

---

### Step 2: Enable Static Website Hosting

1. Click on the bucket name **`fcaj-media-portal-web-2026`**.
2. Navigate to the **Properties** tab.
3. Scroll down to the **Static website hosting** card $\rightarrow$ Click **Edit**.
4. Configure the hosting options:
   - **Static website hosting**: Select **Enable**.
   - **Hosting type**: Select **Host a static website**.
   - **Index document**: Enter:
     ```text
     index.html
     ```
   - **Error document**: Enter `index.html` (optional fallback).
5. Click **Save changes**.
6. Scroll back down to **Static website hosting** and copy the **Bucket website endpoint**:
   ```text
   [http://fcaj-media-portal-web-2026.s3-website-ap-southeast-2.amazonaws.com](http://fcaj-media-portal-web-2026.s3-website-ap-southeast-2.amazonaws.com)
   ```
7. Store this URL on your scratchpad.

**Checkpoint:** The Static website hosting card reflects the **Enabled** state along with an active HTTP endpoint.
![Web-Hosting3](/images/5-Workshop/5.8-Web-Hosting/S3-web-3.png)
---

### Step 3: Apply Public Read Bucket Policy

With Block Public Access turned off, assign the policy authorizing anonymous read access to website assets:

1. Open the **Permissions** tab of `fcaj-media-portal-web-2026`.
2. Scroll to the **Bucket policy** panel $\rightarrow$ Click **Edit**.
3. In the JSON editor, paste the following policy definition *(adjust bucket name in the Resource field if customized)*:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObjectForWebsite",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::fcaj-media-portal-web-2026/*"
    }
  ]
}
```

4. Click the orange **Save changes** button in the bottom-right corner.

**Checkpoint:** The bucket displays a red **Public** badge, confirming anonymous read permissions are active for web distribution.
![Web-Hosting4](/images/5-Workshop/5.8-Web-Hosting/S3-web-4.png)
![Web-Hosting5](/images/5-Workshop/5.8-Web-Hosting/S3-web-5.png)
---

### Step 4: Prepare the `index.html` Frontend Application

1. Open your local code editor (e.g., **Visual Studio Code**).
2. Open the `index.html` file created during Section 5.2.
3. Replace the entire contents of the file with the following complete implementation:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AWS Media Vault - Portal</title>
  <style>
    * { box-sizing: border-box; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
      max-width: 700px;
      margin: 40px auto;
      padding: 0 20px;
      background-color: #f4f6f8;
      color: #161e2e;
    }
    .header {
      text-align: center;
      margin-bottom: 30px;
    }
    .header h1 {
      margin: 0;
      color: #232f3e;
      font-size: 26px;
    }
    .header p {
      color: #68707f;
      margin-top: 8px;
    }
    .card {
      background: #ffffff;
      padding: 24px;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.08);
      margin-bottom: 24px;
      border: 1px solid #e1e4ea;
    }
    .card h2 {
      margin-top: 0;
      font-size: 18px;
      color: #232f3e;
      border-bottom: 2px solid #f4f6f8;
      padding-bottom: 10px;
    }
    .form-group {
      margin-top: 15px;
    }
    label {
      display: block;
      margin-bottom: 6px;
      font-weight: 600;
      font-size: 14px;
    }
    input[type="file"], input[type="text"] {
      width: 100%;
      padding: 10px;
      border: 1px solid #d5dbdb;
      border-radius: 4px;
      font-size: 14px;
      background: #fafafa;
    }
    input[type="text"]:focus {
      outline: none;
      border-color: #ec7211;
      background: #ffffff;
    }
    button {
      display: inline-block;
      width: 100%;
      padding: 12px;
      margin-top: 15px;
      border: none;
      border-radius: 4px;
      font-size: 15px;
      font-weight: bold;
      cursor: pointer;
      transition: background 0.2s ease-in-out;
    }
    .btn-upload {
      background-color: #ec7211;
      color: #ffffff;
    }
    .btn-upload:hover {
      background-color: #eb5f07;
    }
    .btn-download {
      background-color: #161e2e;
      color: #ffffff;
    }
    .btn-download:hover {
      background-color: #0a0f18;
    }
    .status {
      margin-top: 15px;
      padding: 12px;
      border-radius: 4px;
      font-size: 13px;
      line-height: 1.5;
      display: none;
      white-space: pre-wrap;
      word-break: break-all;
    }
    .status.info {
      background-color: #ebf8ff;
      border: 1px solid #bee3f8;
      color: #2b6cb0;
    }
    .status.success {
      background-color: #f0fff4;
      border: 1px solid #c6f6d5;
      color: #22543d;
    }
    .status.error {
      background-color: #fff5f5;
      border: 1px solid #fed7d7;
      color: #9b2c2c;
    }
    .status a {
      color: #2b6cb0;
      font-weight: bold;
      text-decoration: underline;
    }
  </style>
</head>
<body>

  <div class="header">
    <h1>AWS Media Vault Portal</h1>
    <p>Serverless Media Ingestion, Event Processing & Retrieval on AWS</p>
  </div>

  <!-- SECTION 1: UPLOAD -->
  <div class="card">
    <h2>1. Upload Media (Images / Videos)</h2>
    <div class="form-group">
      <label for="fileInput">Choose a file from your workstation:</label>
      <input type="file" id="fileInput" accept="image/*,video/*">
    </div>
    <button class="btn-upload" onclick="uploadMedia()">Upload to S3 Cloud</button>
    <div id="uploadStatus" class="status"></div>
  </div>

  <!-- SECTION 2: DOWNLOAD -->
  <div class="card">
    <h2>2. Secure Media Retrieval (Download)</h2>
    <div class="form-group">
      <label for="downloadFileName">Enter target filename to retrieve:</label>
      <input type="text" id="downloadFileName" placeholder="e.g., test.png or video.mp4">
    </div>
    <button class="btn-download" onclick="downloadMedia()">Generate Download Link</button>
    <div id="downloadStatus" class="status"></div>
  </div>

  <script>
    // =========================================================================
    // CONFIGURE YOUR API GATEWAY ENDPOINT HERE (Must end with /media)
    // =========================================================================
    const API_ENDPOINT = "[https://xxxxxx.execute-api.ap-southeast-2.amazonaws.com/prod/media](https://xxxxxx.execute-api.ap-southeast-2.amazonaws.com/prod/media)";

    async function uploadMedia() {
      const fileInput = document.getElementById('fileInput');
      const status = document.getElementById('uploadStatus');

      if (!fileInput.files.length) {
        alert('Please select an image or video file before uploading!');
        return;
      }

      const file = fileInput.files[0];
      status.style.display = 'block';
      status.className = 'status info';
      status.innerText = `[1/2] Requesting S3 Presigned URL for "${file.name}"...`;

      try {
        // Step 1: Request Presigned URL (PUT) from API Gateway
        const response = await fetch(API_ENDPOINT, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            action: 'get_upload_url',
            fileName: file.name
          })
        });

        if (!response.ok) {
          throw new Error(`API Gateway request failed (HTTP ${response.status})`);
        }

        const data = await response.json();
        if (!data.uploadUrl) {
          throw new Error('Failed to acquire valid uploadUrl from server.');
        }

        // Step 2: Stream binary file directly to S3 via Presigned URL
        status.innerText = `[2/2] Streaming binary payload directly to Amazon S3 Data Bucket...`;
        const uploadResponse = await fetch(data.uploadUrl, {
          method: 'PUT',
          body: file
        });

        if (uploadResponse.ok) {
          status.className = 'status success';
          status.innerText = `Upload Succeeded!\n\n` +
            `• Filename: ${file.name}\n` +
            `• Payload Size: ${(file.size / 1024).toFixed(2)} KB\n` +
            `• S3 Event: Automatically triggered Lambda metadata extraction to DynamoDB.\n` +
            `• Alert: Check your email inbox for the Amazon SNS delivery summary.`;
        } else {
          throw new Error(`Amazon S3 rejected upload (HTTP ${uploadResponse.status})`);
        }
      } catch (error) {
        status.className = 'status error';
        status.innerText = `Execution Error:\n${error.message}`;
      }
    }

    async function downloadMedia() {
      const fileNameInput = document.getElementById('downloadFileName');
      const status = document.getElementById('downloadStatus');
      const fileName = fileNameInput.value.trim();

      if (!fileName) {
        alert('Please enter a target filename to download!');
        return;
      }

      status.style.display = 'block';
      status.className = 'status info';
      status.innerText = `Requesting secure retrieval link for "${fileName}"...`;

      try {
        // Request Presigned URL (GET) from API Gateway
        const response = await fetch(API_ENDPOINT, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            action: 'get_download_url',
            fileName: fileName
          })
        });

        if (!response.ok) {
          throw new Error(`API Gateway error (HTTP ${response.status})`);
        }

        const data = await response.json();
        if (!data.downloadUrl) {
          throw new Error('Could not generate download link for the requested file.');
        }

        status.className = 'status success';
        status.innerHTML = `Secure download link generated (Expires in 5 minutes):<br><br>` +
          `<a href="${data.downloadUrl}" target="_blank" download>` +
          `[ Click here to download: ${fileName} ]` +
          `</a>`;
      } catch (error) {
        status.className = 'status error';
        status.innerText = `Retrieval Error:\n${error.message}`;
      }
    }
  </script>
</body>
</html>
```

4. **Update the API Endpoint:** Locate line 182 and update `API_ENDPOINT` with your deployed Invoke URL from Section 5.7:
   ```javascript
   const API_ENDPOINT = "https://<api-id>[.execute-api.ap-southeast-2.amazonaws.com/prod/media](https://.execute-api.ap-southeast-2.amazonaws.com/prod/media)";
   ```
5. Save the file (**Ctrl + S** or `Cmd + S`).

---

### Step 5: Upload `index.html` to S3 Web Bucket

1. In the **AWS Management Console**, navigate to **Amazon S3**.
2. Click on the bucket **`fcaj-media-portal-web-2026`**.
3. Under the **Objects** tab, click the orange **Upload** button.
4. Click **Add files** $\rightarrow$ Select `index.html` from your local machine.
5. Scroll to the bottom and click **Upload**.

**Checkpoint:** The banner displays `Upload succeeded`. The object `index.html` is listed under the bucket objects.

---

### Step 6: Validate Static Website Endpoint

1. Navigate to the **Properties** tab of `fcaj-media-portal-web-2026`.
2. Scroll to the **Static website hosting** card at the bottom.
3. Click the **Bucket website endpoint** URL directly (e.g., `http://fcaj-media-portal-web-2026.s3-website-ap-southeast-2.amazonaws.com`).
4. The **AWS Media Vault Portal** renders in your browser.

**Checkpoint:** The web portal renders with operational Upload and Download modules without script errors or broken assets.

---

## 3. Expected Outcomes

- S3 Bucket `fcaj-media-portal-web-2026` provisioned and configured for Static Website Hosting in Sydney `ap-southeast-2`.
- Public read access policy applied to web assets without compromising private media storage in the data bucket.
- Single-page client web application (`index.html`) deployed, integrating Fetch API calls against API Gateway and S3 Presigned URLs.
- Web layer staged for real-time telemetry observation and proactive alerting with Amazon CloudWatch in Section 5.9.