---
title: "Blog 1"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Differences Between an Internship Project and a Production-Ready One: The CourShare Architectural Story

When I started building CourShare - a learning and teaching platform sharing a single wallet for both roles - my initial goal was very simple: understand how AWS operates and apply it to a meaningful product, not just isolated lab exercises. After setting up the microservices architecture running on ECS Fargate, I asked myself: "If this architecture had to handle real load, real users, and real money flowing through Stripe - would it be up to standard?"

The short answer: not quite - and that turned out to be a more interesting discovery than I expected.

### Current Architecture: Good for Learning, Not Ready for Production

Currently, CourShare consists of 5 microservices (Identity, Course, Enrollment, Learning, Payment) running on ECS Fargate behind an Application Load Balancer, communicating internally via Cloud Map. Each service has its own database schema, authenticated via Cognito. Videos are uploaded directly to S3 via presigned URLs, then transcoded to HLS format by a worker listening to SQS, with the entire pipeline automatically deployed through GitHub Actions -> ECR -> ECS.

Overall, this is a step in the right direction: clear domain separation, asynchronous processing for heavy tasks (video transcoding), and utilizing a CDN for static content and streaming. However, when looking through the lens of the AWS Well-Architected Framework, I realized one thing: most of the design decisions lean heavily towards cost optimization at the expense of reliability and security. This is perfectly reasonable for a student/internship project, but would be risky if pushed straight to production.

### Three Things I Would Change First If Moving CourShare to Production

#### 1. Replace the NAT Instance with a NAT Gateway
Currently, all private subnets (where microservices run) access the internet through a t4g.nano NAT Instance - a small, cheap EC2 instance, but a single point of failure. If that instance experiences an issue, all services in the private subnets lose internet connection: ECS cannot pull new images, the Payment Service cannot call the Stripe API, and the video worker cannot download/upload to S3.

AWS NAT Gateway solves this exact issue. It is a managed, highly available service within an Availability Zone (AZ), meaning I don't need to patch it or monitor its uptime myself. The trade-off is a much higher cost than a NAT Instance, but in return, I get peace of mind when handling real traffic.

#### 2. Add Perimeter Security: WAF, Secrets Manager, and Data Encryption
Currently, CourShare's API Gateway is directly exposed to the internet without AWS WAF in front. This means there is no layer filtering malicious requests, and no rate limiting to prevent brute-force attacks or bot scraping. Furthermore, database connection strings and Stripe secret keys should be moved to AWS Secrets Manager instead of raw environment variables. Encryption at rest must also be verified for both RDS and the S3 Media Bucket since video lectures and user data are critical assets to protect.

### Further Improvements for Systems with Active Traffic

Additionally, I have brainstormed some longer-term enhancements:
* **Auto Scaling for ECS Services** - to automatically add tasks when there is a surge in students uploading or watching videos, and scale back down during off-peak hours (providing better load handling while saving cost when traffic is low).
* **Cache Layer (e.g., Redis/ElastiCache)** - for highly-read, rarely-changed data like featured courses to offload direct queries from RDS.
* **Infrastructure as Code (Terraform/CDK)** - currently, the infrastructure is manually configured via the AWS Console. As the system scales, IaC becomes the only way to manage changes safely with code reviews and rollbacks, avoiding relying on "remembering what I clicked in the console".
* **CloudWatch Alarms + Dashboards** - moving beyond just capturing logs. The goal is to detect issues as they happen rather than waiting for user reports.
* **S3 Lifecycle Policy for Media Bucket** - large raw videos can accumulate high storage costs. Implementing lifecycle rules to transition them to cheaper storage options (like Glacier) or performing periodic cleanups will keep storage costs manageable.

### Key Takeaways

The best part of reviewing CourShare’s architecture through the Well-Architected Framework isn't compiling a "must-fix" checklist. It's realizing that every architectural decision is an intentional trade-off. There is no "absolutely correct" architecture - only one that fits the current stage and its constraints (budget, learning goals, real-world user scale). Choosing a NAT Instance over a NAT Gateway or Single-AZ over Multi-AZ is not a mistake; it's a sensible choice for a demo, provided we understand the trade-offs and have a roadmap to upgrade when needed.

This is exactly why I find this exercise worth sharing: it is not to claim "my architecture is perfect," but to showcase the thought process of asking: "If this had to run under real-world load, what would break first?" - which, to me, is the most exciting part of learning AWS.

---
*View original post at:* [AWS Study Group - FB Group](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2229312141167079/?rdid=LvHCFBM57fzgRzFP#)