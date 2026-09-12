---
title : "Prerequisites"
date : 2026-09-25
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

### Objectives

Ensure the reader completes AWS Free Tier account registration, configures initial access standards, standardizes on the target AWS Region, sets up local developer tooling, and stages the project codebase before deploying the **AWS Media Vault** architecture.

---

## 1. AWS Account Registration & Preparation

This workshop uses services entirely eligible under the **AWS Free Tier**. If you do not possess an active AWS account, prepare the following items:

- **International Payment Card (Visa/Mastercard)**: With at least 1.00 USD available balance for initial authorization validation (automatically refunded by AWS).
- **Phone Number & Personal Email**: Required for identity verification via OTP during sign-up and for receiving critical operational incident alerts via Amazon SNS.

### AWS Account Sign-Up Steps:

1. Navigate to the sign-up portal: [https://aws.amazon.com/free/](https://aws.amazon.com/free/) and click **Create a Free Account**.
2. Enter your Root user email address and define an AWS account name.
3. Complete email verification using the verification code sent to your inbox.
4. Establish a strong Root account password (minimum 8 characters including uppercase, lowercase, numbers, and symbols).
5. Provide contact details (select **Personal** account type).
6. Enter credit/debit card billing details (card number, expiration date, cardholder name, CVV/CVC).
7. Complete identity verification via SMS text message OTP.
8. On the Support Plan selection screen, choose **Basic Support - Free**.
9. Sign in to the AWS Management Console once the account activation completes.
![Registration Home](/images/2-Proposal/proposal1.png)

---

## 2. Required Tools & Local Environment

The architecture is built entirely on serverless primitives, executed via the **AWS Management Console (Web UI)** without requiring complex Infrastructure as Code (IaC) engines.

Prepare the following software packages locally:

- **Modern Web Browser**: Google Chrome, Mozilla Firefox, or Microsoft Edge (updated to recent builds to ensure full Console compatibility and Fetch API support for the web portal).
- **Visual Studio Code (or preferred IDE)**: Used for updating API endpoints in `index.html` and editing `lambda_function.py`.
- **Git**: For cloning the repository and managing code revisions.
- **Python (v3.10+)**: Helpful for syntax checking and running local Boto3 verification scripts if needed.
- **Active Personal Email Inbox**: Required to receive and confirm the Amazon SNS notification subscription.

---

## 3. Step-by-Step Instructions

**Log in to AWS Console:** Sign in to the AWS Management Console using your provisioned account credentials. Ensure the Region selector in the top-right header is set to **ap-southeast-2 (Sydney)**.
![AWS Console page](/images/2-Proposal/proposal2.png)

**Checkpoint:** Verify that the active Region displays **Asia Pacific (Sydney) ap-southeast-2** prior to launching any services, ensuring compatibility between S3, Lambda, API Gateway, and DynamoDB.

**Verify Local Tooling:** Launch a Terminal (macOS/Linux) or Command Prompt / PowerShell (Windows) to verify that local runtime utilities are operational:

```bash
git --version
python --version   # or python3 --version
code --version     # checks Visual Studio Code installation (optional)
```

**Checkpoint:** All commands return valid installed version numbers for Git and Python.

**Clone & Stage Project Codebase:** Clone the project repository from GitHub to your workstation or create a dedicated working directory:

```bash
git clone ...
cd aws-media-vault
code .
```

Verify that the project directory includes the following core files:
- `index.html`: Client web portal for uploading and downloading media objects directly via S3 Presigned URLs.
- `lambda_function.py`: Serverless compute handler written in Python 3.12 utilizing the Boto3 SDK.

**Checkpoint:** The project opens successfully in Visual Studio Code, with all assets staged for IAM security configuration in the subsequent chapter.

---

## 4. Expected Outcomes

- Successfully registered and signed in to the AWS Management Console under the Free Tier targeting the **ap-southeast-2 (Sydney)** Region.
- Fully staged local workspace (Browser, VS Code, Git, Python).
- An active email inbox verified and ready for Amazon SNS operational alerts.
- Project repository cloned and reviewed prior to cloud infrastructure provisioning.