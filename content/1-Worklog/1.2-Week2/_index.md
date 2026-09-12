---
title: "Worklog Week 2"
date: 2026-08-10
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:
* Perform deep-dive research and hands-on practice with core AWS services: IAM, Amazon EC2, and Amazon S3.
* Apply the Principle of Least Privilege across IAM Users, User Groups, Roles, and custom Policies.
* Provision, network, secure, and remotely administer Amazon EC2 virtual servers.
* Manage unstructured object storage in Amazon S3, enforce Bucket Policies, and host static web content.
* Attach an IAM Instance Profile to EC2 to enable keyless authentication against Amazon S3.

### Tasks planned for this week:

| Day | Tasks | Start Date | End Date | Documentation |
| --- | --- | --- | --- | --- |
| Mon | - Study IAM Policy JSON document schema (Version, Statement, Effect, Action, Resource, Condition).<br>- Contrast Identity-based Policies, Resource-based Policies, and Permission Boundaries.<br>- Create a `Developers` IAM Group, craft customized least-privilege policies, and verify user boundary enforcement. | 10/08/2026 | 10/08/2026 | https://docs.aws.amazon.com/IAM/latest/UserGuide/ |
| Tue | - Study the AWS Nitro System virtualization framework and EC2 instance types (General Purpose, Compute/Memory/Storage Optimized).<br>- Generate RSA Key Pairs (`.pem` / `.ppk`), configuring strict permissions via `chmod 400 key.pem`.<br>- Launch an Amazon Linux 2023 EC2 instance (`t3.micro` Free Tier) inside Availability Zone `ap-southeast-1a`. | 11/08/2026 | 11/08/2026 | https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ |
| Wed | - Configure Security Group rules (Inbound: port 22 SSH restricted to My IP, port 80 HTTP open to 0.0.0.0/0).<br>- Establish an SSH connection: `ssh -i key.pem ec2-user@<public-ip>`.<br>- Execute `dnf update -y`, install the Apache Web Server (`httpd`), author a custom `index.html`, and verify HTTP connectivity via browser. | 12/08/2026 | 12/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| Thu | - Explore Amazon S3 architecture: Buckets, Object Keys, Metadata, and Storage Classes (Standard, IA, Glacier).<br>- Provision a globally unique S3 Bucket and enable Bucket Versioning for data integrity.<br>- Perform object synchronization operations via AWS CLI: `aws s3 cp`, `aws s3 sync`, and `aws s3 ls`. | 13/08/2026 | 13/08/2026 | https://docs.aws.amazon.com/AmazonS3/latest/userguide/ |
| Fri | - Disable S3 `Block Public Access`, craft a public read Bucket Policy for `s3:GetObject`.<br>- Enable Static Website Hosting and access web content via the S3 Website Endpoint URL.<br>- Attach an IAM Role with `AmazonS3ReadOnlyAccess` to the EC2 instance and confirm secret-less access via `aws s3 ls` from within the virtual machine. | 14/08/2026 | 14/08/2026 | https://cloudjourney.awsstudygroup.com/ |

### Week 2 Achievements:
* **Completion Rate:** 100%.
* **Theoretical Knowledge:**
  * Understood RBAC vs ABAC models in AWS IAM, and the critical distinction between permanent user credentials and temporary STS credentials issued to IAM Roles.
  * Mastered the EC2 Instance lifecycle stages and pricing structures (On-Demand, Spot, Savings Plans).
  * Grasped Amazon S3 Strong Read-After-Write Consistency.
* **Practical Skills:**
  * Configured and hardened an Apache Web Server running on Amazon Linux 2023.
  * Hosted a resilient static website using Amazon S3 storage infrastructure.
  * Eliminated long-term credential vulnerabilities by provisioning EC2 IAM Instance Profiles.