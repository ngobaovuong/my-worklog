---
title: "Worklog Week 6"
date: 2026-09-07
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:
* Design and implement highly available, fault-tolerant web architectures across multiple Availability Zones.
* Master traffic distribution using Elastic Load Balancing (ELB), focusing on the Application Load Balancer (ALB).
* Implement automated compute elasticity using EC2 Auto Scaling Groups (ASG).
* Integrate ALB Target Groups with Auto Scaling dynamic tracking policies to absorb fluctuating traffic spikes.

### Tasks planned for this week:

| Day | Tasks | Start Date | End Date | Documentation |
| --- | --- | --- | --- | --- |
| Mon | - Study AWS Load Balancer families: Application Load Balancer (Layer 7), Network Load Balancer (Layer 4), and Gateway Load Balancer.<br>- Examine ALB architecture: Listeners, Routing Rules, Target Groups, and Health Checks.<br>- Deploy two EC2 instances across two distinct AZs configured with dynamic hostname output scripts. | 07/09/2026 | 07/09/2026 | https://docs.aws.amazon.com/elasticloadbalancing/latest/application/ |
| Tue | - Create an HTTP (port 80) Target Group with configured health check thresholds for `/index.html`.<br>- Register both EC2 instances into the Target Group.<br>- Provision an internet-facing Application Load Balancer across two Public Subnets.<br>- Verify round-robin load distribution across both web instances via the ALB DNS name. | 08/09/2026 | 08/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| Wed | - Study Auto Scaling Group (ASG) capacity metrics: Minimum, Maximum, and Desired Capacity.<br>- Build an EC2 Launch Template defining AMI, `t3.micro` instance type, Security Groups, IAM Roles, and automated User Data web installation scripts.<br>- Contrast modern Launch Templates with deprecated Launch Configurations. | 09/09/2026 | 09/09/2026 | https://docs.aws.amazon.com/autoscaling/ec2/userguide/ |
| Thu | - Launch an Auto Scaling Group referencing the new Launch Template.<br>- Distribute the ASG across two Private Subnets and attach it directly to the ALB Target Group.<br>- Define a dynamic Target Tracking Scaling Policy targeting an average CPU utilization of 50%.<br>- Configure cooldown periods to prevent scaling thrashing. | 10/09/2026 | 10/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| Fri | - Test self-healing resilience: manually terminate an EC2 instance, observing the ASG detect the capacity shortfall and automatically launch a replacement instance.<br>- Execute CPU stress benchmarking: observe CloudWatch Alarms trigger a scale-out event expanding instances from 2 to 4.<br>- Relieve stress load and monitor graceful scale-in instance termination.<br>- Update Hugo documentation and review deliverables. | 11/09/2026 | 11/09/2026 | https://cloudjourney.awsstudygroup.com/ |

### Week 6 Achievements:
* **Completion Rate:** 100%.
* **Theoretical Knowledge:**
  * Mastered multi-tier architecture design (Presentation Tier on ALB/Public Subnets, Application Tier on ASG/Private Subnets).
  * Understood Path-based and Host-based HTTP routing rules supported by ALB.
  * Grasped the principles of elasticity, horizontal scaling, and automated self-healing systems.
* **Practical Skills:**
  * Deployed a production-grade, highly available multi-AZ infrastructure.
  * Configured active health check mechanisms to remove failing nodes from routing pools automatically.
  * Successfully validated dynamic scale-out and scale-in policies driven by real-time compute load.