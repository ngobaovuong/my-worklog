---
title: "Worklog Week 4"
date: 2026-08-24
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:
* Master Serverless architectural patterns and event-driven computing with AWS Lambda.
* Understand the internal Lambda execution lifecycle (Cold Starts vs. Warm Starts, microVM isolation).
* Develop backend logic in Python and Node.js, managing memory allocations, execution timeouts, and environment variables.
* Configure asynchronous and synchronous triggers using Amazon S3 Event Notifications and Amazon API Gateway.
* Inspect execution traces, metrics, and application logs using Amazon CloudWatch Logs.

### Tasks planned for this week:

| Day | Tasks | Start Date | End Date | Documentation |
| --- | --- | --- | --- | --- |
| Mon | - Study the Serverless computing paradigm: zero server management, automatic scaling from 0 to thousands of concurrent executions, sub-second pay-per-use billing.<br>- Compare use-cases and cost trade-offs: EC2 vs ECS vs AWS Lambda.<br>- Analyze Lambda core components: Handlers, Event Payloads, and Context Objects. | 24/08/2026 | 24/08/2026 | https://docs.aws.amazon.com/lambda/latest/dg/ |
| Tue | - Create a Lambda function using the Python 3.12 runtime via the Console.<br>- Configure an IAM Execution Role with `AWSLambdaBasicExecutionRole` policy.<br>- Benchmark runtime performance: adjust memory from 128 MB to 512 MB (evaluating proportional vCPU scaling) and configure a 15-second timeout.<br>- Write test handlers parsing JSON parameters and run executions with mock events. | 25/08/2026 | 25/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| Wed | - Build an automated file-processing pipeline: Provision an Amazon S3 source bucket.<br>- Grant the Lambda Execution Role `s3:GetObject` access permissions.<br>- Establish an S3 Event Trigger bound to `s3:ObjectCreated:*` events.<br>- Upload sample files to S3, verifying automated function invocation and metadata parsing. | 26/08/2026 | 26/08/2026 | https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html |
| Thu | - Introduction to Amazon API Gateway (REST APIs vs. HTTP APIs).<br>- Build an HTTP API serving as a lightweight webhook gateway handling POST requests.<br>- Configure Lambda Proxy Integration, mapping inbound JSON HTTP payloads into the function.<br>- Execute end-to-end integration testing via Postman and cURL. | 27/08/2026 | 27/08/2026 | https://docs.aws.amazon.com/apigateway/latest/developerguide/ |
| Fri | - Investigate Lambda Cold Starts: container initialization overhead and mitigation techniques (Provisioned Concurrency, slim code packages).<br>- Analyze CloudWatch Log Streams: review `Duration`, `Billed Duration`, `Memory Size`, and `Max Memory Used` performance metrics.<br>- Document serverless findings and submit Week 4 worklog. | 28/08/2026 | 28/08/2026 | https://cloudjourney.awsstudygroup.com/ |

### Week 4 Achievements:
* **Completion Rate:** 100%.
* **Theoretical Knowledge:**
  * Mastered Event-Driven Architectures (EDA) and decoupled microservice patterns.
  * Differentiated synchronous invocations (API Gateway) from asynchronous event streams (S3 notifications).
  * Understood the resource allocation model and cost optimization techniques of AWS Lambda.
* **Practical Skills:**
  * Implemented automated backend processing triggered by S3 bucket events.
  * Built and deployed a production-ready serverless API endpoint using API Gateway and AWS Lambda.