---
title : "Monitoring & Logging"
date : 2024-01-01 
weight : 11
chapter : false
pre : " <b> 5.11. </b> "
---

# Centralized Monitoring & Logging with Amazon CloudWatch

In a distributed microservices architecture, tracking system health and debugging errors can be challenging if logs are fragmented across multiple ephemeral container tasks. We implement **Centralized Logging** by routing all container stdout and stderr to **Amazon CloudWatch Logs**, allowing real-time monitoring and log querying from a unified dashboard.

## 1. The `awslogs` Log Driver Mechanism
AWS ECS Fargate natively supports the `awslogs` log driver. This driver automatically captures standard output (stdout) and standard error (stderr) streams from the application processes inside the containers and routes them directly to CloudWatch Logs.

## 2. Configuring Logs in the Task Definition
Within the ECS Task Definition JSON for each microservice, configure the `logConfiguration` block:

```json
"logConfiguration": {
    "logDriver": "awslogs",
    "options": {
        "awslogs-group": "/ecs/courshare-microservices",
        "awslogs-region": "us-east-1",
        "awslogs-stream-prefix": "identity-service",
        "awslogs-create-group": "true"
    }
}
```

* **awslogs-group:** The unified log group name for the entire application, e.g., `/ecs/courshare-microservices`.
* **awslogs-stream-prefix:** The prefix for the log streams to easily distinguish source logs from different services (e.g., `identity-service`, `course-service`, `video-worker`, etc.).

## 3. Querying Logs via the CloudWatch Console
1. Navigate to the **CloudWatch Console** -> **Logs** -> **Log groups**.
2. Select the `/ecs/courshare-microservices` log group.
3. You will see a list of individual log streams corresponding to the active/historical Fargate container tasks.
4. Leverage **CloudWatch Logs Insights** to run fast queries across all microservice log streams:
   ```sql
   fields @timestamp, @message
   | filter @message like /Error/ or @message like /Exception/
   | sort @timestamp desc
   | limit 100
   ```

<!-- TODO: chèn screenshot - [Amazon CloudWatch Logs Groups console listing the /ecs/courshare-microservices log group] -->

<!-- TODO: chèn screenshot - [CloudWatch Logs Insights editor running a search query to filter error and exception logs from all container streams] -->
