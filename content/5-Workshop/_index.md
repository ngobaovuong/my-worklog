---
title: "Workshop"
date: 2026-09-13
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Deploying AWS Media Vault on AWS

#### Overview

In this hands-on workshop, we will architect, build, and deploy **AWS Media Vault** — an automated, cloud-native serverless media processing, storage, and incident alert platform on AWS.

The architecture integrates 7 core AWS services: **Amazon S3** (Private Object Storage & Static Web Hosting), **AWS Lambda** (Serverless Compute), **Amazon API Gateway** (Managed REST API with CORS), **Amazon DynamoDB** (High-performance NoSQL Metadata Store), **Amazon SNS** (Push Notification Engine), **Amazon CloudWatch** (Log Analytics, Metrics & Proactive Alarms), and **AWS IAM** (Identity and Least-Privilege Access Management).

Throughout this practical workshop, you will be guided step-by-step through configuring IAM permissions, provisioning backend storage, scripting dual-purpose Lambda compute functions, wiring API Gateway with S3 Presigned URLs, hosting a frontend on S3, establishing operational observability, executing fault-injection tests, and cleanly tearing down resources.

#### Table of Contents

1. [Workshop Overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequisite/)
3. [IAM Security Configuration](5.3-IAM-Setup/)
4. [Storage & Database Setup](5.4-Storage-Database/)
5. [SNS Notification Setup](5.5-SNS-Notification/)
6. [Compute Logic with AWS Lambda](5.6-Lambda-Compute/)
7. [API Gateway Integration](5.7-API-Gateway/)
8. [Static Web Hosting on S3](5.8-Web-Hosting/)
9. [Monitoring & Alarms with CloudWatch](5.9-Monitoring/)
10. [System Testing & Fault Injection](5.10-Testing/)
11. [Resource Cleanup](5.11-Cleanup/)