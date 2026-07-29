---
title: "Workshop"
date: 2026-07-28
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Deploying the CourShare Platform on AWS Cloud

In this chapter, we will walk through the step-by-step practical implementation and deployment of the entire CourShare platform on AWS infrastructure. The sections are organized in a logical sequence, starting from core networking, container deployments, database provisioning, user security, media storage, async workers, CI/CD automation, monitoring, and resource cleanup.

#### Workshop Content

1. [Architectural Introduction and System Overview](5.1-Workshop-overview/)
2. [Deploy Environment Prerequisites](5.2-Prerequisite/)
3. [Setting up VPC Networking Infrastructure](5.3-Networking-setup/)
4. [Deploying Microservices with AWS ECS Fargate & ALB](5.4-Deploying-ECS-Fargate/)
5. [Deploying the Amazon RDS Database Layer](5.5-Database-layer/)
6. [User Identity and Authentication with Amazon Cognito](5.6-Authentication-Cognito/)
7. [Integrating S3, CloudFront CDN & Secure Video Uploads](5.7-Storage-CDN-Video-Upload/)
8. [Decoupling the System via Asynchronous SQS & Workers](5.8-Asynchronous-Processing/)
9. [Deploying the Payment Service & Stripe-Integrated Wallet](5.9-Payment-Service-Wallet/)
10. [Automating Deployments with CI/CD via GitHub Actions](5.10-CI-CD-Pipeline/)
11. [Centralized Monitoring & Logging with Amazon CloudWatch](5.11-Monitoring-Logging/)
12. [Resource Cleanup to Avoid Unwanted Charges](5.12-Cleanup/)