---
title: "Week 1 Worklog"
date: 2026-06-15
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Week 1 Objectives:

* Research cloud-native application development models and compare Monolithic vs. Microservices architectures.
* Design the overview system architecture diagram for the CourShare project.
* Analyze and define the roles of core AWS cloud services in the system layout.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | - Study and compare traditional Monolithic architecture with Microservices running in Cloud environments <br> - Evaluate independent scaling, high availability, and fault isolation | 15/06/2026 | 15/06/2026 | Cloud-native patterns & Microservices guide |
| 2 | - Define and design the CourShare system consisting of 5 component services (Identity, Course, Payment, Enrollment, Learning Services) | 16/06/2026 | 16/06/2026 | CourShare System Design Document |
| 3 | - Design the data flow through API Gateway (Single Entry Point) via a centralized Lambda Authorizer authorization mechanism | 17/06/2026 | 17/06/2026 | AWS API Gateway & Lambda Authorizer Documentation |
| 4 | - Design the request forwarding flow from API Gateway to Private Subnets via Internal Application Load Balancer (ALB) | 18/06/2026 | 18/06/2026 | AWS ALB Developer Guide |
| 5 | - Draft the initial Entity Relationship Diagram (ERD) for the microservices databases | 19/06/2026 | 19/06/2026 | Database Design & ERD Best Practices |

### Week 1 Achievements:

* Completed the overall system architecture design and decomposed functions into 5 microservices running on separate ports (8081, 8082, 8083, 8084, 8085).
* Defined the centralized security routing through a Custom Authorizer before forwarding traffic to the private network.
* Completed the initial Entity Relationship Diagram (ERD) for the microservices, defining the specific roles of the cloud services to be utilized.
* **Initial account infrastructure**: Successfully registered the AWS account, set up an IAM Admin User with multi-factor authentication (MFA) enabled, and installed AWS CLI on the local machine to verify connectivity.

![IAM Console](/images/1-WorkLog/iam-console.png)
*AWS IAM Console page showing the administrator account configured with MFA.*

![AWS CLI Terminal](/images/1-WorkLog/terminal.png)
*Verifying AWS CLI connectivity using `aws sts get-caller-identity` command from local terminal.*

