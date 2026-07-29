---
title : "Deploying ECS Fargate & ALB"
date : 2024-01-01 
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

# Deploying Microservices with AWS ECS Fargate & ALB

We will package CourShare's 5 microservices and 2 workers into Docker images, push them to Amazon ECR, run them as serverless tasks inside the AWS ECS Fargate Cluster, and configure public routing via the Application Load Balancer.

## Step 1: Create Amazon ECR Repositories
1. Navigate to the **Amazon ECR** Console -> **Repositories** -> **Create repository**.
2. Create 7 private repositories corresponding to the services:
   - `courshare-identity-service`
   - `courshare-course-service`
   - `courshare-enrollment-service`
   - `courshare-learning-service`
   - `courshare-payment-service`
   - `courshare-video-worker`
   - `courshare-notification-worker`

## Step 2: Build & Push Images to ECR (Manual Test)
1. Authenticate your local Docker client to the ECR Registry:
   ```bash
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
   ```
2. Navigate into each service's code directory (e.g., Identity Service) and build the Docker image:
   ```bash
   docker build -t courshare-identity-service .
   docker tag courshare-identity-service:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/courshare-identity-service:latest
   docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/courshare-identity-service:latest
   ```
   *(Repeat these commands for all other microservices and workers).*

## Step 3: Create the Application Load Balancer (ALB)
1. Navigate to **EC2 Console** -> **Load Balancers** -> **Create Load Balancer** -> Select **Application Load Balancer**.
2. Name: `courshare-alb`.
3. Network Mapping:
   - VPC: `courshare-vpc`
   - Subnets: Select `courshare-public-subnet-1` and `2`.
4. Security Groups: Select `courshare-alb-sg`.
5. Listeners and Routing:
   - Configure 1 Listener on port `80` (HTTP) directing to a default target group.
6. Click **Create load balancer**.

## Step 4: Author Task Definitions
For each microservice/worker, create a Task Definition on ECS:
1. Select launch type: **AWS Fargate**.
2. Task size: CPU: `0.25 vCPU`, Memory: `0.5 GB` (Minimal specifications to save budget).
3. Task Role & Task Execution Role: Choose `ecsTaskExecutionRole` (endowed with policies to pull ECR images and publish logs to CloudWatch).
4. Container Definitions:
   - Name: e.g., `identity-service`.
   - Image URI: `<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/courshare-identity-service:latest`.
   - Port Mapping: container ports `8080` (Identity) / `8081` (Course) / `8082` (Enrollment) / `8083` (Learning) / `8084` (Payment).

## Step 5: Create Target Groups & Configure Routing
1. Create 5 separate Target Groups corresponding to the 5 microservices (Target type: **IP**, Protocol: **HTTP**, Port: service container port).
2. Go to the **ALB Listener Rules** of Port 80, and add path-based rules:
   - Path `/auth/**` -> Forward to `tg-identity-service`.
   - Path `/courses/**` -> Forward to `tg-course-service`.
   - Path `/enrollments/**` -> Forward to `tg-enrollment-service`.
   - Path `/learning/**` -> Forward to `tg-learning-service`.
   - Path `/payment/**` -> Forward to `tg-payment-service`.

## Step 6: Initialize ECS Services & Service Discovery
1. Create the **ECS Cluster** named `courshare-cluster`.
2. Launch ECS Services for the 5 microservices:
   - Launch type: **Fargate**.
   - Service Name: e.g., `identity-service`.
   - Number of tasks: `1`.
   - Network Settings:
     - Subnets: Select `courshare-private-subnet-1` and `2`.
     - Security Group: Select `courshare-ecs-tasks-sg`.
     - Auto-assign public IP: **Disabled**.
   - Load Balancing: Select `courshare-alb`, and link it to the appropriate Target Group.
3. **Configure Service Discovery (Amazon Cloud Map):**
   - Enable **Service Discovery**.
   - Namespace: Select `courshare.local`.
   - Service discovery name: e.g., `identity-service`.
   - Now services can invoke each other internally via: `http://identity-service.courshare.local:8080`.
4. Deploy the ECS Services for `VideoWorker` and `NotificationWorker` (no Load Balancer is needed since they are background daemons consuming from SQS).

<!-- TODO: chèn screenshot - [Amazon ECR Console listing the 7 private repositories for the project] -->

<!-- TODO: chèn screenshot - [AWS ECS Console showing all services in the cluster with Active status and Running Tasks = 1] -->

<!-- TODO: chèn screenshot - [AWS Route 53 Private Hosted Zone showing the internal DNS records created automatically by Cloud Map] -->
