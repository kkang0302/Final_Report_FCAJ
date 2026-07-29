---
title: "Week 5 Worklog"
date: 2026-07-13
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:

* Deploy a centralized API Gateway routing system for the CourShare project.
* Build a centralized user authentication mechanism using an AWS Lambda Custom Authorizer with JWT signatures.
* Link the API Gateway with the Private network using a VPC Link and an Internal Application Load Balancer (ALB).

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | - Build the AWS Lambda Custom Authorizer in Node.js to decode JWT, verify signatures, and extract user details | 13/07/2026 | 13/07/2026 | AWS Lambda Authorizer SDK |
| 2 | - Configure HTTP API Gateway as the single entry point for all client requests and set up CORS policies | 14/07/2026 | 14/07/2026 | HTTP API Gateway Setup |
| 3 | - Establish a VPC Link to connect the API Gateway to the Internal Application Load Balancer (ALB) inside the Private Subnets | 15/07/2026 | 15/07/2026 | VPC Link Integration |
| 4 | - Configure Parameter Mapping at the API Gateway to inject user context from the Lambda Authorizer into downstream request Headers | 16/07/2026 | 16/07/2026 | API Gateway Parameter Mapping Guide |
| 5 | - Set up Listener Rules on the Internal ALB to route traffic based on paths (`/auth*`, `/courses*`, `/checkout*`, `/enrollments*`, `/progress*`...) to their respective Target Groups | 17/07/2026 | 17/07/2026 | ALB Routing Rules Guide |

### Week 5 Achievements:

* All requests passing through the API Gateway are centrally authenticated and authorized using the Lambda Authorizer.
* The API Gateway successfully forwards identity details (`X-User-Id`, `X-User-Email`, `X-User-Roles`) to the backends in the Private Subnets via the VPC Link and the Internal ALB.
* Path-based routing rules operate accurately, routing requests to the appropriate microservices.

![ALB Rules](/images/1-WorkLog/ALB.png)
*Path-based listener routing rules configured on the Internal Application Load Balancer.*

