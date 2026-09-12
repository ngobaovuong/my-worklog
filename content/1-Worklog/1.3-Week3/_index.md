---
title: "Worklog Week 3"
date: 2026-08-17
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:
* Master isolated cloud networking architecture using Amazon Virtual Private Cloud (Amazon VPC).
* Implement Classless Inter-Domain Routing (CIDR) subnetting to carve out Public and Private Subnets.
* Build custom Route Tables, attach Internet Gateways (IGW), and analyze NAT Gateway architectures.
* Implement a defense-in-depth networking model by configuring Security Groups (Stateful) and Network ACLs (Stateless).

### Tasks planned for this week:

| Day | Tasks | Start Date | End Date | Documentation |
| --- | --- | --- | --- | --- |
| Mon | - Study cloud networking topologies, RFC 1918 private address spaces, and CIDR math.<br>- Provision a Custom VPC with a `10.0.0.0/16` CIDR block (providing 65,536 private IP addresses).<br>- Enable DNS Resolution and DNS Hostnames attributes on the custom VPC. | 17/08/2026 | 17/08/2026 | https://docs.aws.amazon.com/vpc/latest/userguide/ |
| Tue | - Analyze the 5 reserved IP addresses per subnet (.0, .1, .2, .3, .255).<br>- Create Public Subnet A (`10.0.1.0/24`) in `ap-southeast-1a` and Public Subnet B (`10.0.2.0/24`) in `ap-southeast-1b`.<br>- Create Private Subnet A (`10.0.10.0/24`) in `ap-southeast-1a` and Private Subnet B (`10.0.20.0/24`) in `ap-southeast-1b`. | 18/08/2026 | 18/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| Wed | - Provision and attach an Internet Gateway (IGW) to the Custom VPC.<br>- Create a custom Public Route Table, define the default route `0.0.0.0/0 -> IGW`.<br>- Associate Public Subnet A and B with the Public Route Table.<br>- Maintain the VPC Main Route Table (`10.0.0.0/16 local`) strictly for isolated Private Subnets. | 19/08/2026 | 19/08/2026 | https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html |
| Thu | - Deploy an EC2 instance in Public Subnet A with auto-assigned Public IPv4.<br>- Deploy a second EC2 instance inside Private Subnet A (isolated from public ingress).<br>- Configure a Bastion Host (Jump Box) on the Public Subnet to securely SSH into the Private EC2 machine.<br>- Study NAT Gateway routing to allow outbound internet access for private instances. | 20/08/2026 | 20/08/2026 | https://cloudjourney.awsstudygroup.com/ |
| Fri | - Create and analyze custom Network Access Control Lists (NACLs): analyze top-to-bottom rule number precedence and stateless behavior.<br>- Contrast Security Groups (ENI-level, stateful) against Network ACLs (subnet-level, stateless).<br>- Test specific IP blocking by applying an inbound NACL Deny rule and observe traffic termination. | 21/08/2026 | 21/08/2026 | https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html |

### Week 3 Achievements:
* **Completion Rate:** 100%.
* **Theoretical Knowledge:**
  * Understood the multi-AZ enterprise VPC topology required for geographic redundancy.
  * Clearly differentiated Internet Gateway functionality from managed NAT Gateway services.
  * Mastered the distinction between Stateful firewalling (Security Groups) and Stateless packet inspection (NACLs).
* **Practical Skills:**
  * Designed and deployed a complete two-tier multi-AZ VPC from scratch.
  * Implemented Bastion Host architectures for secure fleet management within private subnets.