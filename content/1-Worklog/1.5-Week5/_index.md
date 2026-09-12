---
title: "Worklog Week 5"
date: 2026-08-31
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:
* Master cloud observability using Amazon CloudWatch (Metrics, Logs, and Alarms).
* Configure custom CloudWatch Dashboards and install the CloudWatch Unified Agent for OS-level telemetry.
* Configure Amazon Simple Notification Service (Amazon SNS) to publish real-time alert notifications to email subscribers.
* Leverage AWS CloudTrail for governance, compliance auditing, and root-cause analysis (RCA) of API activity.

### Tasks planned for this week:

| Day | Tasks | Start Date | End Date | Documentation |
| --- | --- | --- | --- | --- |
| Mon | - Study the 3 pillars of observability: Metrics, Logs, and Traces.<br>- Analyze standard EC2 hypervisor metrics (`CPUUtilization`, `DiskReadBytes`, `NetworkIn`, `StatusCheckFailed`).<br>- Differentiate Basic Monitoring (5-minute frequency, free) from Detailed Monitoring (1-minute intervals, billable). | 31/08/2026 | 31/08/2026 | https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ |
| Tue | - Create a CloudWatch Custom Dashboard featuring Line, Stacked, and Gauge widgets.<br>- Install and configure the CloudWatch Unified Agent on an Amazon Linux EC2 instance.<br>- Collect advanced operating-system metrics hidden from the hypervisor: Memory (RAM) utilization and Disk Space usage. | 01/09/2026 | 01/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| Wed | - Explore pub/sub messaging patterns with Amazon Simple Notification Service (SNS).<br>- Create an SNS Standard Topic named `DevOps-Alerts` and subscribe an email endpoint.<br>- Construct a CloudWatch Metric Alarm: transition to ALARM state when `CPUUtilization >= 75%` for 2 consecutive evaluation periods.<br>- Bind the ALARM action directly to the SNS Topic. | 02/09/2026 | 02/09/2026 | https://docs.aws.amazon.com/sns/latest/dg/ |
| Thu | - Install the Linux `stress` benchmark package on the EC2 machine: execute `stress --cpu 2 --timeout 300s`.<br>- Observe the metric surge on the CloudWatch Dashboard and watch the alarm transition from `OK` to `ALARM`.<br>- Verify automated email delivery dispatched through Amazon SNS.<br>- Confirm automatic recovery back to `OK` status once the stress test ends. | 03/09/2026 | 03/09/2026 | https://cloudjourney.awsstudygroup.com/ |
| Fri | - Study AWS CloudTrail: differentiate Management Events, Data Events, and Insights Events.<br>- Create an enterprise Multi-Region Trail writing encrypted audit logs to an Amazon S3 bucket.<br>- Use CloudTrail Event History to investigate simulated security incidents (trace actor IAM identity, source IP, API calls, and modified Security Groups).<br>- Wrap up Week 5 documentation. | 04/09/2026 | 04/09/2026 | https://docs.aws.amazon.com/awscloudtrail/latest/userguide/ |

### Week 5 Achievements:
* **Completion Rate:** 100%.
* **Theoretical Knowledge:**
  * Understood the boundary between hypervisor-level metrics and OS-level telemetry.
  * Grasped the publish/subscribe decoupling model implemented by Amazon SNS.
  * Learned compliance, auditing, and threat investigation workflows powered by AWS CloudTrail.
* **Practical Skills:**
  * Configured the CloudWatch Unified Agent to report custom system memory and disk utilization.
  * Designed an end-to-end automated alerting pipeline via CloudWatch Alarms and SNS.
  * Performed forensic audit investigations using CloudTrail event logs.