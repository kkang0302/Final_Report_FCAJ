---
title: "Week 4 Worklog"
date: 2026-07-06
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

* Design and automate the entire network, security, and database infrastructure on AWS via Terraform (Infrastructure as Code).
* Build an EC2 NAT Instance to optimize operations costs instead of a standard AWS NAT Gateway.
* Set up a shared database system using Amazon RDS PostgreSQL in an isolated network partition.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | - Write Terraform configuration files (`main.tf`) defining a Virtual Private Cloud (VPC) with IP range `10.0.0.0/16` | 06/07/2026 | 06/07/2026 | Terraform AWS Provider Docs |
| 2 | - Establish a network structure consisting of Public, Private, and Database Subnets across 2 Availability Zones (`ap-southeast-1a`, `ap-southeast-1b`) | 07/07/2026 | 07/07/2026 | AWS VPC Subnet Design Guidelines |
| 3 | - Provision an EC2 NAT Instance (`t3.micro`, Amazon Linux 2023) in a Public Subnet and write `user_data` script to enable package forwarding and route Private Subnets | 08/07/2026 | 08/07/2026 | NAT Instance configuration guides |
| 4 | - Provision Amazon RDS PostgreSQL (`db.t4g.micro`) inside isolated Database Subnets | 09/07/2026 | 09/07/2026 | Terraform RDS Resource Guide |
| 5 | - Configure Security Group for port `5432` to only allow internal traffic from ECS and NAT Instance; verify with `terraform apply` | 10/07/2026 | 10/07/2026 | AWS Security Group Rules |

### Week 4 Achievements:

* Successfully set up a secure AWS infrastructure in the Singapore region entirely using Terraform.
* The custom NAT Instance functions properly, letting resources in Private Subnets access the Internet without public exposure, saving significant costs.
* The RDS PostgreSQL database is running stably, securely isolated in its database subnet.

![VPC Map](/images/1-WorkLog/VPC.png)
*VPC Resource Map visual diagram showing subnet layouts on AWS.*

![VPC Subnets](/images/1-WorkLog/Subnet.png)
*List of subnets (Public, Private, Database) created via Terraform on the AWS Console.*

![VPC Route Tables](/images/1-WorkLog/Route-table.png)
*Route tables mapping traffic directions through the Internet Gateway or the custom NAT Instance.*

![EC2 NAT Instance](/images/1-WorkLog/EC2.png)
*The custom NAT Instance running on an EC2 virtual machine under Running state.*

![RDS Database](/images/1-WorkLog/RDS.png)
*Amazon RDS PostgreSQL database instance configured under Available state.*

