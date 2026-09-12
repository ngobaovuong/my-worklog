---
title: "Worklog Week 1"
date: 2026-08-03
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Week 1 Objectives:
* Attend the internship orientation, understand workplace rules, working workflows, and the Cloud Foundation training curriculum.
* Master cloud computing fundamentals (IaaS, PaaS, SaaS) and AWS Global Infrastructure components (Regions, Availability Zones, Edge Locations).
* Successfully register an AWS Free Tier account and implement foundational security hardening baselines.
* Install, configure, and utilize the AWS Command Line Interface (AWS CLI v2) on the local workstation.
* Achieve a perfect score (200/200 points) across all 5 initial onboarding tasks.

### Tasks planned for this week:

| Day | Tasks | Start Date | End Date | Documentation |
| --- | --- | --- | --- | --- |
| Mon | - Attend the internship orientation session, meet assigned mentors, and receive learning guidelines.<br>- Study foundational AWS cloud concepts and the AWS Shared Responsibility Model.<br>- Compare on-premises legacy environments with the scalability of public cloud platforms. | 03/08/2026 | 03/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| Tue | - Register an AWS Free Tier account.<br>- Enforce Multi-Factor Authentication (MFA via Virtual Authenticator) on the Root user account.<br>- Permanently delete Root user access keys to prevent high-privilege credential leakage.<br>- Set up AWS Budgets and a CloudWatch Billing Alarm to trigger notifications at a $1.00 threshold. | 04/08/2026 | 04/08/2026 | https://docs.aws.amazon.com/accounts/latest/reference/ |
| Wed | - Deep dive into AWS Global Infrastructure: latency benchmarking across Singapore (ap-southeast-1), Tokyo (ap-northeast-1), and Sydney (ap-southeast-2).<br>- Execute Task 1: Explore AWS Management Console navigation and Region switching.<br>- Execute Task 2: Provision a dedicated IAM Admin User, set up IAM Groups, and discontinue daily root account usage. | 05/08/2026 | 05/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| Thu | - Install AWS CLI v2 across local operating environments.<br>- Generate Access Key ID & Secret Access Key for the IAM User; run `aws configure` setting output to `json` and default region to `ap-southeast-1`.<br>- Execute Task 3: Validate identity configurations via `aws sts get-caller-identity`.<br>- Execute Task 4: Query available regions using `aws ec2 describe-regions`. | 06/08/2026 | 06/08/2026 | https://docs.aws.amazon.com/cli/latest/userguide/ |
| Fri | - Execute Task 5: Evaluate access rights and evaluate policies with the IAM Policy Simulator.<br>- Complete all 5 onboarding tasks with a 200/200 perfect score.<br>- Draft Week 1 Hugo Markdown worklog and attend weekly progress review with the host supervisor. | 07/08/2026 | 07/08/2026 | https://cloudjourney.awsstudygroup.com/ |

### Week 1 Achievements:
* **Completion Rate:** 100%.
* **Theoretical Knowledge:**
  * Comprehended the 6 core advantages of Cloud Computing (trade capital expense for variable expense, benefit from massive economies of scale, stop guessing capacity, increase speed and agility, stop spending money running data centers, go global in minutes).
  * Understood the Shared Responsibility Model: AWS handles security "of the cloud" (hardware, facility, network isolation), while customers manage security "in the cloud" (IAM policies, OS patching, firewall configurations, data encryption).
* **Practical Skills:**
  * Hardened the AWS account in compliance with the CIS AWS Foundations Benchmark.
  * Gained proficiency in executing cloud infrastructure queries via AWS CLI v2.
  * Successfully attained 200/200 points on the onboarding curriculum.