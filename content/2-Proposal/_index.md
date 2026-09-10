---
title: "Proposal"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Second-Hand Marketplace

## A Cloud-Native Second-Hand Marketplace on AWS

---

# 1. Executive Summary

Second-Hand Marketplace is a cloud-based web application that enables users to buy and sell second-hand products through a centralized online marketplace. The platform provides user authentication, product management, category management, image uploading, product searching, shopping cart, checkout, and order management while utilizing AWS managed services to ensure scalability, availability, security, and simplified deployment.

The application is developed using **Node.js**, **Express.js**, **MongoDB Atlas**, and **EJS**. The application is containerized using **Docker** and deployed on **Amazon ECS Fargate** behind an **Application Load Balancer (ALB)**. Docker images are stored in **Amazon ECR**, uploaded product images are stored in **Amazon S3**, and **AWS CodeBuild** automatically builds and deploys the latest application whenever new source code is pushed to GitHub.

The deployment environment also utilizes **Amazon Route 53** for domain management, **AWS Certificate Manager (ACM)** for HTTPS encryption, **Amazon CloudWatch** for monitoring, **AWS IAM** for access control, and **Amazon VPC** for secure networking. The architecture provides automated deployment, centralized storage, simplified management, and a scalable cloud infrastructure suitable for small and medium-sized e-commerce applications.

---

# 2. Problem Statement

## Current Problem

Many second-hand marketplaces rely on social media platforms or manually managed websites, making product management inefficient and difficult to maintain. Product images are often stored locally, deployments require manual updates, and scaling the application becomes increasingly difficult as the number of users grows.

Traditional deployment methods also increase downtime, require additional operational effort, and make application maintenance more complicated whenever new features or bug fixes are released.

## Solution

The proposed solution develops a cloud-native second-hand marketplace platform using AWS managed services.

Users can register accounts, log in securely, upload products with images, browse products by category, search products, manage shopping carts, place orders, and manage their own product listings through a web application.

Application data is stored in **MongoDB Atlas**, while uploaded product images are stored in **Amazon S3**.

The application is containerized using Docker and deployed on **Amazon ECS Fargate**. Whenever source code is pushed to GitHub, **AWS CodeBuild** automatically builds a Docker image, pushes it to **Amazon ECR**, and deploys the latest version to Amazon ECS.

HTTPS communication is secured using **AWS Certificate Manager (ACM)** and the application is accessible through a custom domain configured using **Amazon Route 53**.

## Benefits

The proposed architecture provides several benefits:

- Simplified application deployment.
- Automated CI/CD pipeline.
- Scalable cloud infrastructure.
- Secure HTTPS communication.
- Reliable cloud storage.
- Simplified application maintenance.
- Reduced operational effort.
- Easy future scalability.

---

# 3. Solution Architecture

The application follows a cloud-native container architecture deployed on AWS managed services.

## Solution Architecture

![System Architecture](/images/2-Proposal/system_architecture.png)

## AWS Services Utilized

- Amazon VPC
- AWS IAM
- Amazon ECS Fargate
- Amazon ECR
- Amazon S3
- Application Load Balancer (ALB)
- Amazon Route 53
- AWS Certificate Manager (ACM)
- AWS CodeBuild
- Amazon CloudWatch
- MongoDB Atlas

## Component Design

### Frontend

- HTML
- CSS
- JavaScript
- EJS Template Engine

### Backend

- Node.js
- Express.js
- Express Session
- Multer
- AWS SDK for JavaScript

### Database

- MongoDB Atlas

### Image Storage

- Amazon S3

### Container Platform

- Docker
- Amazon ECS Fargate

### Deployment Pipeline

GitHub

↓

AWS CodeBuild

↓

Amazon ECR

↓

Amazon ECS Fargate

---

# 4. Technical Implementation

## Implementation Phases

The project was implemented through the following phases:

- Research AWS cloud architecture and deployment strategy.
- Design the overall marketplace system architecture.
- Develop the backend using Node.js and Express.js.
- Configure MongoDB Atlas for cloud database storage.
- Integrate Amazon S3 for product image storage.
- Containerize the application using Docker.
- Push Docker images to Amazon ECR.
- Deploy Docker containers on Amazon ECS Fargate.
- Configure Application Load Balancer.
- Configure Amazon Route 53 and AWS Certificate Manager (ACM).
- Configure AWS CodeBuild for automated build and deployment.
- Monitor the application using Amazon CloudWatch.
- Perform system testing and deploy the production environment.

## Technical Requirements

### Programming Languages

- JavaScript
- HTML
- CSS

### Frameworks

- Express.js
- EJS

### Database

- MongoDB Atlas

### Cloud Services

- Amazon VPC
- AWS IAM
- Amazon ECS Fargate
- Amazon ECR
- Amazon S3
- Application Load Balancer (ALB)
- Amazon Route 53
- AWS Certificate Manager (ACM)
- AWS CodeBuild
- Amazon CloudWatch

### Development Tools

- Visual Studio Code
- Git
- GitHub
- Docker Desktop
- MongoDB Compass
---

# 5. Roadmap & Milestones

The project was completed through the following implementation phases.

### Phase 1 – Project Planning

- Analyze system requirements.
- Design the overall system architecture.
- Design the MongoDB database structure.
- Prepare the development environment.

### Phase 2 – Application Development

- Develop user authentication.
- Develop customer functions.
- Develop shop management functions.
- Develop administrator functions.
- Develop product management.
- Develop order management.

### Phase 3 – Cloud Integration

- Configure MongoDB Atlas.
- Integrate Amazon S3 for image storage.
- Test cloud storage connectivity.

### Phase 4 – Containerization

- Create Dockerfile.
- Build Docker Image.
- Test the Docker container locally.

### Phase 5 – AWS Deployment

- Push Docker Image to Amazon ECR.
- Deploy the application to Amazon ECS Fargate.
- Configure Application Load Balancer.
- Configure Amazon Route 53.
- Configure AWS Certificate Manager (ACM).

### Phase 6 – CI/CD

- Connect GitHub repository.
- Configure AWS CodeBuild.
- Automate application deployment.

### Phase 7 – Monitoring & Testing

- Configure Amazon CloudWatch.
- Perform functional testing.
- Verify application deployment.
- Fix deployment issues.

### Phase 8 – Project Completion

- Deploy the production environment.
- Complete documentation.
- Demonstrate the completed project.

---

# 6. Budget Estimation

## Infrastructure Cost Estimate

| Service | Estimated Cost |
|----------|----------------|
| Amazon ECS Fargate | ~$0.25/month |
| Amazon S3 (Storage & Requests) | ~$0.15/month |
| Amazon ECR | ~$0.03/month |
| AWS CodeBuild | ~$0.05/month |
| Application Load Balancer | ~$0.10/month |
| Amazon CloudWatch | ~$0.02/month |
| **Total Estimate** | **~$0.60 USD/month** |

### Cost Control Guidelines

- **AWS Budgets:** Automated alerts when costs exceed **$5.00** and **$10.00**.
- **Amazon ECR Lifecycle Policy:** Automatically remove unused Docker images.
- **AWS CodeBuild:** Trigger builds only when code is pushed to the GitHub repository.
- **Post-demo Cleanup:** Remove ECS services, ECR images, unused S3 objects, Application Load Balancer, CloudWatch alarms, ACM certificates, and Route 53 hosted zones after project completion to avoid unnecessary charges.

---

# 7. Risk Assessment

## Risk Matrix

- Amazon ECS deployment failure.
- MongoDB Atlas connectivity issues.
- Amazon S3 upload failures.
- AWS CodeBuild build failures.
- Route 53 DNS configuration issues.
- HTTPS certificate configuration issues.
- Unexpected AWS service charges.

## Mitigation Strategies

- Enable Amazon CloudWatch monitoring.
- Configure AWS Budgets alerts.
- Version Docker images using Amazon ECR.
- Perform regular MongoDB Atlas backups.
- Apply IAM least privilege policies.
- Verify Route 53 DNS records before deployment.
- Validate ACM certificate status before enabling HTTPS.

## Contingency Plans

- Roll back to the previous Docker image.
- Redeploy the previous Amazon ECS Task Definition.
- Restore MongoDB Atlas backups.
- Redeploy through AWS CodeBuild.
- Reconfigure Route 53 DNS records if necessary.
- Reissue the ACM certificate when validation fails.

---

# 8. Expected Results

## Technical Results

The completed project will provide:

- A fully containerized cloud-native second-hand marketplace platform.
- Automatic CI/CD deployment using GitHub and AWS CodeBuild.
- Reliable image storage using Amazon S3.
- Scalable container deployment using Amazon ECS Fargate.
- Secure HTTPS communication using AWS Certificate Manager (ACM).
- Custom domain management using Amazon Route 53.
- Load balancing using Application Load Balancer.
- Centralized cloud database using MongoDB Atlas.
- Resource monitoring using Amazon CloudWatch.
- Secure access management using AWS IAM.

## Business Value

The project demonstrates the practical implementation of cloud computing, containerization, and DevOps practices using AWS managed services.

The cloud-native architecture simplifies deployment, reduces operational effort, improves scalability, and provides a reliable foundation for future expansion.

Future enhancements may include online payment integration, recommendation systems, notification services, analytics dashboards, and microservice-based architecture while maintaining high availability and operational efficiency.