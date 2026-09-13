---
title: "Worklog Week 7"
date: 2026-09-07
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:
* Master containerization fundamentals with Docker (Images, Containers, Layers, Multi-stage builds).
* Host container artifacts securely using Amazon Elastic Container Registry (Amazon ECR).
* Learn container orchestration fundamentals with Amazon Elastic Container Service (Amazon ECS).
* Analyze compute options: ECS EC2 Launch Type vs. serverless AWS Fargate.
* Build, publish, and deploy a containerized web service on AWS Fargate coupled with an Application Load Balancer.

### Tasks planned for this week:

| Day | Tasks | Start Date | End Date | Documentation |
| --- | --- | --- | --- | --- |
| Mon | - Review container fundamentals: compare virtual machines (guest OS hypervisors) with lightweight container isolation (shared Linux kernel).<br>- Author an optimized Dockerfile packaging a Node.js/Python web application utilizing an `alpine` base image.<br>- Build and validate containers locally: `docker build -t web-app:v1 .` and `docker run -d -p 8080:80 web-app:v1`. | 14/09/2026 | 14/09/2026 | https://docs.docker.com/get-started/ |
| Tue | - Explore Amazon Elastic Container Registry (ECR) for secure image lifecycle management.<br>- Create a private ECR repository named `cloud-demo-app` with Image Scanning on push enabled.<br>- Authenticate Docker CLI against ECR: `aws ecr get-login-password \| docker login ...`.<br>- Tag and push the application container image to the ECR registry. | 15/09/2026 | 15/09/2026 | https://docs.aws.amazon.com/AmazonECR/latest/userguide/ |
| Wed | - Study AWS ECS architecture: Clusters, Task Definitions, Services, and Tasks.<br>- Contrast ECS on EC2 (managing underlying cluster instances) with AWS Fargate (fully serverless container compute).<br>- Distinguish the two ECS IAM roles: Task Execution Role (pulling images from ECR, pushing logs) vs Task Role (application runtime access to AWS APIs). | 16/09/2026 | 16/09/2026 | https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ |
| Thu | - Author an ECS Task Definition: set launch type compatibility to `FARGATE`, assign 0.5 vCPU and 1 GB memory, specify ECR image URI.<br>- Configure the `awslogs` log driver to stream container stdout/stderr into CloudWatch Log Groups.<br>- Provision a dedicated ECS Fargate Cluster. | 17/09/2026 | 17/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| Fri | - Create an ECS Service: configure Desired Tasks = 2 using the `awsvpc` network mode across VPC subnets.<br>- Restrict container Security Groups strictly to inbound HTTP traffic.<br>- Connect the ECS Service to an Application Load Balancer Target Group.<br>- Validate task health, query the application over the ALB DNS name, and document Week 7 results. | 18/09/2026 | 18/09/2026 | https://cloudjourney.awsstudygroup.com/ |

### Week 7 Achievements:
* **Completion Rate:** 100%.
* **Theoretical Knowledge:**
  * Understood the end-to-end containerized application lifecycle.
  * Evaluated the operational benefits of serverless container orchestration via AWS Fargate.
  * Mastered the `awsvpc` networking mode where each task receives its own dedicated Elastic Network Interface (ENI).
* **Practical Skills:**
  * Packaged and hardened lightweight production-grade Docker images.
  * Managed secure image registries and vulnerability scanning in Amazon ECR.
  * Deployed a production-ready, auto-recovering containerized web service on AWS ECS Fargate.