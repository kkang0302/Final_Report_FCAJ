---
title: "Event 2"
date: 2026-07-11
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# Summary Report: "Cloud Architecture Battle 2"

### Event Objectives

- Master advanced enterprise architecture concepts, including disaster recovery, hybrid cloud networking, and security.
- Practice fast-paced architectural decision-making under intense competition.
- Solidify theoretical knowledge on AWS best practices and the Well-Architected Framework.

### Event Format

Building on the first event, the tournament escalated in challenge:
1. Two teams competed in the final round of the tournament.
2. The teams answered highly complex multiple-choice questions detailing architectural failure scenarios, migration paths, and compliance standards.
3. Questions featured real-world production incidents, requiring deep conceptual synthesis.
4. Final points determined the overall tournament champion.

### Key Cloud Architecture Knowledge Gained

#### 1. Disaster Recovery (DR) Strategies at Scale
- Deep dive into defining Recovery Point Objective (RPO) and Recovery Time Objective (RTO) metrics.
- Comparing and evaluating DR mechanisms on AWS: Backup and Restore, Pilot Light, Warm Standby, and Multi-Site Active-Active designs.
- Implementing cross-region replication strategies using Aurora Global Database and Route 53 Application Recovery Controller to automate regional failover.

#### 2. Enterprise Hybrid Cloud Connectivity
- Designing secure and dedicated network paths between corporate on-premises data centers and AWS.
- Selecting the appropriate setup for AWS Direct Connect (DX), AWS Transit Gateway, and IPSec VPN tunnels to balance cost, performance, and redundancy.

#### 3. Advanced Security & Compliance
- Implementing data protection using AWS Key Management Service (KMS) with Customer Managed Keys (CMK) and proper key policies.
- Protecting web applications from common exploits and DDoS attacks using AWS WAF, AWS Shield, and Amazon CloudFront.
- Aligning infrastructure designs with strict security standards like PCI-DSS and HIPAA through AWS Artifact and security best practices.

### Guest Speaker Sharing Sessions

At the end of the event, we had insightful sharing sessions from two industry specialists:

#### 1. DevSecOps Engineer Sharing: AWS Security Agent
- **Focus:** Securing EC2 instances and container runtimes.
- **Key Concepts:** Practical guide on deploying and managing the **AWS Systems Manager (SSM) Agent** and the **Amazon Inspector Agent** to continuously scan for vulnerabilities, verify patch compliance, and ensure real-time threat detection. Emphasized integrating security agents in the CI/CD pipeline to achieve automated security scanning.

#### 2. NOC Engineer Sharing: Service Level Agreements (SLA)
- **Focus:** Practical operations, monitoring uptime, and availability calculation.
- **Key Concepts:** Deep dive into how SLA is calculated (e.g., "three nines" vs "four nines" uptime) and its financial/legal impacts. Learned how NOC teams monitor live architectures to detect anomalies before SLAs are breached, using Amazon CloudWatch metrics and Service Health dashboards.

### Event Photos

![Advanced Cloud Architecture Battle Discussion](/images/4-EventParticipated/4.2-Event2/Messenger_creation_B6B2B4BE-ECB8-4DE6-B98E-F8461010D4B1.png)

