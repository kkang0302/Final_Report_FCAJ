---
title : "Prerequisites"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

# Deploy Environment Prerequisites

Before beginning the steps to deploy CourShare on AWS, you need to prepare the following accounts, privileges, and local developer tooling.

## 1. AWS Account & Privileges
* You need an active **AWS account** (leveraging the AWS Free Tier is highly recommended).
* Avoid using the Root account. Create an **IAM User** with AdministratorAccess (or specific roles scoped for VPC, ECS, RDS, S3, Cognito, SQS, CloudWatch, and ECR).
* Enable Multi-Factor Authentication (MFA) for your administrator IAM account.

## 2. Developer Tooling on Client Machine
Download and install the following developer tools on your local machine:

* **AWS CLI (v2):** Command-line interface to interact with AWS resources from your local terminal.
  - Download from: [AWS CLI Installation Guide](https://aws.amazon.com/cli/)
  - Configure the AWS CLI using:
    ```bash
    aws configure
    ```
    Input your Access Key ID, Secret Access Key, default region (e.g., `us-east-1`), and output format (`json`).
* **Docker Desktop:** Required to package, run, and push Docker container images of the microservices to Amazon ECR.
  - Download from: [Docker Desktop](https://www.docker.com/products/docker-desktop/)
* **Git:** For source control and connection to GitHub to set up the CI/CD pipeline.
* **Node.js (v18+) & Java JDK 17:** To run and develop the microservices locally (Spring Boot and Express Node.js).
* **Database Client:** E.g., **DBeaver** or **pgAdmin** to connect to and inspect the RDS PostgreSQL database.

## 3. CourShare Service Repositories
Ensure you have cloned all 6 repositories of the project:
* Identity Service (Java Spring Boot)
* Course Service (Java Spring Boot)
* API Gateway (Spring Cloud Gateway)
* Enrollment Service (Node.js/Express)
* Learning Service (Node.js/Express)
* Payment Service (Node.js/Express)

<!-- TODO: chèn screenshot - [Terminal window running `aws --version` and `docker --version` verifying successful local tool installations] -->
