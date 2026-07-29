---
title: "Proposal"
date: 2026-07-28
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# CourShare - Integrated Sharing and Online Learning Platform
## Containerized Microservices Architecture running on AWS ECS Fargate

### 1. Executive Summary
**CourShare** is an online learning platform combining the strengths of video sharing models (similar to YouTube) and structured courses (similar to Udemy/Coursera). However, CourShare narrows its scope of content to focus strictly on educational lecture videos, avoiding distractions from multi-topic entertainment content.

The platform is designed to run on the AWS cloud to optimize operational capability, reliability, and scalability. By applying a microservices architecture running on a Serverless Container environment (AWS ECS Fargate), CourShare not only meets the high-performance demands of online learning but also provides a comprehensive engineering environment, utilizing modern AWS services to solve practical problems.

### 2. Problem Statement
* **Current Issues:**
  On popular online learning platforms today, the roles of "student" and "instructor" are often strictly separated. Users must register different types of accounts or navigate complex, separate management interfaces. Crucially, the payment wallets and checkout flows for these two roles are also distinct, causing significant friction when instructors want to use the balance earned from selling their courses to buy and learn other courses on the same platform (they must withdraw money to their bank accounts, pay transaction fees, and deposit it back to purchase another course).

* **CourShare's Solution:**
  CourShare introduces a **Shared Wallet** model for each user, integrated into a single account. Anyone who registers an account can play a dual role: uploading their own courses to share/sell, and finding, purchasing, and studying courses from others.
  - When a student buys your course, the earnings are instantly credited to your internal e-wallet.
  - When you buy someone else's course, the cost is deducted directly from that same wallet balance.
  - This mechanism creates an **internal creator economy**, promoting lifelong learning, minimizing payment friction, and simplifying balance management for individuals or small creators.

* **Benefits and Return on Investment (ROI):**
  - **Technical aspect:** Helps the engineering team practice designing distributed systems, handling eventual consistency between the Payment Service and Enrollment Service, managing asynchronous media workflows, and optimizing AWS infrastructure.
  - **Operational aspect:** Minimizes external payment gateway fees for internal transactions due to the shared wallet mechanism. The system leverages AWS Free Tier and cheap NAT instances to keep development and testing costs extremely low.

### 3. Solution Architecture
The CourShare system architecture is built based on Microservices, running on a Serverless Container platform (ECS Fargate) to ensure independent security between services, automatic scaling, and reduced server management overhead.

![CourShare Architecture](/images/AWS_CourShare.drawio.png)

#### AWS Services Used
* **Amazon CloudFront (CDN):** Distributes static website assets (SPA Frontend) and high-performance HLS video streams globally with low latency.
* **Amazon S3:**
  - *Frontend Web Bucket:* Stores the static React SPA build.
  - *Media Bucket:* Stores raw source videos and transcoded HLS stream files.
* **Amazon API Gateway:** The single entry point for client API requests, routing them to the Application Load Balancer (ALB).
* **Amazon VPC:** Secure, isolated network partition including:
  - *Public Subnet:* Houses the Application Load Balancer and NAT Instance (t4g.nano) to provide outbound internet access to Private Subnets at minimal cost.
  - *Private Subnet:* Houses the AWS ECS Cluster (Fargate) running microservice and worker containers.
  - *Database Subnet:* Completely isolated partition for Amazon RDS PostgreSQL databases.
* **AWS ECS Fargate:** Runs the microservices (Identity, Course, Enrollment, Learning, Payment) and workers (VideoWorker, NotificationWorker) as Docker containers.
* **Amazon Cloud Map:** Service Discovery solution supporting private internal microservice communications within the VPC via DNS names instead of hard-coded IPs.
* **Amazon Cognito:** Manages user identity (Student/Instructor) centrally, providing secure authorization mechanisms.
* **Amazon RDS (PostgreSQL):** Relational database management system for services, configured with separate schemas per service (IdentityDB, CourseDB, EnrollmentDB, LearningDB, PaymentDB) to guarantee database independence.
* **Amazon SQS Queue:** Message queue to handle asynchronous tasks (video transcoding, sending email notifications).
* **Amazon CloudWatch:** Centralized log collection from all ECS Tasks and RDS databases for monitoring and alerting.
* **CI/CD Pipeline (GitHub Actions & Amazon ECR):** Automates Docker image building, pushing to ECR, and deployment updates to ECS Fargate.

#### Component Design
* **Identity Service (Port 8080 - Java Spring Boot):** Manages user profiles and interacts with Amazon Cognito for authentication and JWT token generation.
* **Course Service (Port 8081 - Java Spring Boot):** Manages course details, categories, lessons, and generates Presigned URLs for secure media uploads.
* **Enrollment Service (Port 8082 - Node.js/Express):** Manages students' course enrollment history.
* **Learning Service (Port 8083 - Node.js/Express):** Tracks student learning progress and lesson completion states.
* **Payment Service (Port 8084 - Node.js/Express):** Integrates Stripe API to process real deposits/withdrawals, manage internal wallet balances, and track transaction history.
* **VideoWorker (Node.js):** Consumes SQS messages, downloads raw videos from S3, transcodes them to HLS format (m3u8/ts) using ffmpeg, and uploads them back to S3 for streaming.
* **NotificationWorker (Node.js):** Consumes SQS messages to send email notifications for successful enrollments, wallet updates, or system alerts.

### 4. Technical Implementation
#### Implementation Phases
1. **Research & Architectural Design (Month 1):** Analyze business requirements, design ERDs for each microservice, and draft AWS infrastructure blueprints.
2. **Budget Estimation & Feasibility Analysis (Month 1):** Estimate infrastructure costs using AWS Pricing Calculator, selecting a NAT Instance over NAT Gateway for cost-efficiency.
3. **Infrastructure Provisioning & Tuning (Month 2):** Set up the VPC network, subnets, RDS PostgreSQL instances, and Cognito User Pools.
4. **Development, Containerization & Testing (Month 2 - Month 3):** Implement microservices source code, write Dockerfiles, configure ALB routing, integrate Stripe Webhooks, and build SQS queue workers.
5. **CI/CD & Monitoring Setup (Month 3):** Set up GitHub Actions for automatic ECS Fargate updates and build CloudWatch Dashboards.

#### Technical Requirements
* **Development Environment:** Docker Desktop, Node.js (v18+), Java JDK 17, Prisma CLI, AWS CLI.
* **Microservices Configuration:**
  - Java Spring Boot services use Spring Security integrated with JWT and connect to PostgreSQL via Spring Data JPA.
  - Node.js services use Express and Prisma ORM to query PostgreSQL.
  - Stripe SDK is integrated in the Payment Service to verify and process Stripe Webhook events.

### 5. Timeline & Milestones
* **Week 1 - 2:** Learn AWS Cloud, establish the VPC, and configure secure subnetting.
* **Week 3 - 4:** Containerize services, configure the ECS Fargate Cluster, and set up the ALB to route API calls.
* **Week 5 - 6:** Provision RDS PostgreSQL for the 5 databases; set up the Cognito User Pool for centralized auth.
* **Week 7 - 8:** Set up S3 Frontend and S3 Media buckets. Build the CloudFront CDN distribution. Configure secure Presigned URL flows for direct uploads.
* **Week 9 - 10:** Integrate SQS, build HLS VideoWorker and email NotificationWorker. Complete Payment Service with Stripe integration.
* **Week 11 - 12:** Automate CI/CD pipelines with GitHub Actions and Amazon ECR. Configure monitoring dashboards via CloudWatch. Perform end-to-end integration testing.

### 6. Budget Estimation
The table below estimates monthly infrastructure costs for running CourShare in a testing/demo environment (relying on AWS Free Tier and minimum resource allocations):

| AWS Service | Demo Configuration | Estimated Cost / Month | Notes |
| --- | --- | --- | --- |
| **AWS ECS Fargate** | 5 Services + 2 Workers (0.25 vCPU, 0.5 GB RAM per task) | ~$15.00 | Uses Fargate Spot to save 70% cost |
| **Amazon RDS** | PostgreSQL db.t4g.micro (20 GB SSD) | ~$10.00 | Consolidated multi-schema on 1 instance to save cost |
| **VPC NAT Instance** | 1 instance t4g.nano to route outbound traffic | ~$3.20 | Dramatically cheaper than NAT Gateway (~$32/mo) |
| **Amazon S3 & CDN** | Video storage and CloudFront CDN | ~$2.00 | Depends on video uploads and streaming bandwidth |
| **Amazon Cognito** | Under 50,000 monthly active users | $0.00 | Covered by AWS Free Tier |
| **Amazon SQS & API GW**| Low request frequency | ~$0.50 | Covered by AWS Free Tier |
| **CloudWatch Logs** | Log storage under 5 GB | $0.00 | Covered by AWS Free Tier |
| **Total** | | **~$30.70 / Month** | *Estimated reference. Users can adjust accordingly.* |

### 7. Risk Assessment
* **Risk 1: Video storage and CDN bandwidth costs grow exponentially**
  - *Description:* As more courses and lessons are uploaded, S3 and CloudFront CDN costs can scale up quickly.
  - *Mitigation:* Configure S3 Lifecycle rules to move old videos to Glacier; optimize compression algorithms in the VideoWorker; set video size limits (e.g., max 100MB per video).
* **Risk 2: Data inconsistency between Payment Service and Enrollment Service**
  - *Description:* A failure occurs where the wallet balance is deducted, but the enrollment fails, causing a bad user experience.
  - *Mitigation:* Apply asynchronous transactional patterns (like the Outbox Pattern) or SQS messaging queue with auto-retry mechanisms to guarantee eventual consistency.
* **Risk 3: Copyright infringement and inappropriate content**
  - *Description:* Users uploading copyrighted videos or inappropriate content.
  - *Mitigation:* Run automated moderation using Amazon Rekognition; provide a report button for users, allowing admins to manually review flagged content.
* **Risk 4: High latency in HLS video transcoding**
  - *Description:* Transcoding high-resolution videos takes significant CPU resources, backing up the SQS processing queue.
  - *Mitigation:* Set ECS Auto Scaling to scale VideoWorker tasks up when SQS messages spike; optimize transcoding parameters in ffmpeg.

### 8. Expected Outcomes
* **Robust Infrastructure:** Successfully deploy a containerized microservices infrastructure on AWS ECS Fargate, securely isolated within a private VPC.
* **Functional Platform:** Users can register, deposit funds via Stripe, upload lectures, buy courses using their shared wallet, and stream HLS video seamlessly.
* **Centralized Observability:** Unified logging and error tracking configured via Amazon CloudWatch.
* **Automated CI/CD:** Code changes pushed to GitHub are automatically built, scanned, and deployed to AWS in minutes via ECR and ECS.