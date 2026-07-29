---
title : "Introduction"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

# CourShare Architecture Overview on AWS

In this workshop series, we will build and deploy **CourShare** — an integrated online learning and sharing platform featuring a shared internal wallet economy. The project is hosted on the AWS cloud using a containerized microservices architecture deployed on **AWS ECS Fargate**, ensuring high performance, tight isolation, and dynamic scaling.

## Overall Architecture Diagram

Here is the complete AWS cloud infrastructure layout for the CourShare platform:

![CourShare Architecture Diagram](/images/AWS_CourShare.drawio.png)

## Core Operational Flows

1. **Client Access & CDN:** Users (Students/Instructors) access the Single Page Application (SPA React frontend) stored on the **Amazon S3 Frontend Web Bucket**, distributed globally via the **Amazon CloudFront CDN**.
2. **User Authentication:** Users authenticate directly against the **Amazon Cognito User Pool**. Upon successful login, the client receives a JWT Access Token to attach in the headers of subsequent API requests.
3. **API Routing:** Incoming API requests are directed via the CloudFront CDN to **Amazon API Gateway**, which serves as the entry point and forwards them to the **Application Load Balancer (ALB)**.
4. **Microservices Processing:** The ALB performs path-based routing to direct traffic into the **AWS ECS Fargate Cluster** situated in the Private Subnets:
   - Request path `/auth/**` routes to the **Identity Service** (port 8080) for user profile management.
   - Request path `/courses/**` routes to the **Course Service** (port 8081) for course content management.
   - Request path `/enrollments/**` routes to the **Enrollment Service** (port 8082) for student registration processing.
   - Request path `/learning/**` routes to the **Learning Service** (port 8083) for course progress tracking.
   - Request path `/payment/**` routes to the **Payment Service** (port 8084) to handle wallet balances and Stripe checkout deposits/withdrawals.
5. **Internal Communication (Service Discovery):** Microservices communicate directly with each other in the Private Subnet using local DNS names (e.g., `http://course-service.courshare.local:8081`) via **Amazon Cloud Map**, preventing exposure of private APIs to the public internet.
6. **Data Storage:** Each microservice keeps its data isolated within separate PostgreSQL schemas on a single relational database instance on **Amazon RDS (PostgreSQL)** to optimize costs while preserving data isolation.
7. **Media Transcoding and Asynchronous Workflows:**
   - Instructors request to upload course videos, receiving an **S3 Presigned URL** to upload video files directly to the **Amazon S3 Media Bucket**.
   - An upload event triggers a message dispatch to the **Amazon SQS Queue**.
   - The **VideoWorker** background task consumes SQS messages, downloads the raw videos, transcodes them into HLS format (generating `.m3u8` indexes and `.ts` fragments), and saves them back to S3 for adaptive streaming.
   - The **NotificationWorker** consumes SQS messages to send transactional emails via AWS-integrated mailing mechanisms.
8. **Observability & CI/CD:** Logs from all container tasks are sent to **Amazon CloudWatch Logs**. When code updates are pushed to **GitHub**, the CI/CD pipeline in **GitHub Actions** builds the Docker image, pushes it to **Amazon ECR**, and updates the ECS Service automatically.