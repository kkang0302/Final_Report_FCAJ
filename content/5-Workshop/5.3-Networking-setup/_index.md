---
title : "VPC Networking Setup"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

# Setting up the VPC Networking Infrastructure for CourShare

A robust and secure VPC networking infrastructure is the baseline for deploying a containerized microservices cluster on ECS Fargate. We partition our network into 3 separate subnet tiers across 2 Availability Zones (AZs) to enforce High Availability and data security.

## Step 1: Initialize the VPC
1. Sign in to the AWS Console and navigate to the **VPC** service.
2. Click **Create VPC**.
3. Select **VPC only** (we will configure components manually to understand the network paths).
4. Name: `courshare-vpc`.
5. IPv4 CIDR block: `10.0.0.0/16`.
6. Click **Create VPC**.

## Step 2: Create Subnets (IP Allocation)
We will create 6 subnets across 2 AZs (`us-east-1a` and `us-east-1b`):

1. **2 Public Subnets** (for the ALB and NAT Instance):
   - `courshare-public-subnet-1` | CIDR: `10.0.1.0/24` | AZ: `us-east-1a`
   - `courshare-public-subnet-2` | CIDR: `10.0.2.0/24` | AZ: `us-east-1b`
2. **2 Private Subnets** (for ECS Tasks hosting microservices and workers):
   - `courshare-private-subnet-1` | CIDR: `10.0.10.0/24` | AZ: `us-east-1a`
   - `courshare-private-subnet-2` | CIDR: `10.0.11.0/24` | AZ: `us-east-1b`
3. **2 Database Subnets** (for RDS PostgreSQL):
   - `courshare-db-subnet-1` | CIDR: `10.0.20.0/24` | AZ: `us-east-1a`
   - `courshare-db-subnet-2` | CIDR: `10.0.21.0/24` | AZ: `us-east-1b`

*(Ensure "Auto-assign public IPv4 address" is enabled for both Public Subnets).*

## Step 3: Create the Internet Gateway (IGW)
1. On the VPC Console left-hand menu, select **Internet Gateways** -> **Create Internet Gateway**.
2. Name it `courshare-igw` and click **Create**.
3. Select the created IGW -> **Actions** -> **Attach to VPC** -> Select `courshare-vpc` -> Click **Attach**.

## Step 4: Deploy a NAT Instance (Cost Optimization)
To avoid the high monthly fixed cost of AWS NAT Gateways (approx. $32/month), we deploy a tiny EC2 virtual machine as a NAT proxy:

1. Navigate to **EC2 Console** -> **Launch Instance**.
2. Name: `courshare-nat-instance`.
3. OS: Select **Amazon Linux 2023 AMI**.
4. Instance Type: Select **t4g.nano** (saving budget, costing approx. $3.2/month).
5. Network settings:
   - VPC: `courshare-vpc`
   - Subnet: `courshare-public-subnet-1`
   - Auto-assign Public IP: Enable
6. Select your key pair, configure a Security Group permitting outbound traffic, and enable IP forwarding in the Linux OS.
7. **CRITICAL:** Once the instance is launched, right-click the NAT instance -> **Networking** -> **Change source/destination checking**. Select **Disable** (This is required to allow the instance to forward traffic originating from the Private Subnets).

## Step 5: Configure Route Tables
We create 3 route tables:

1. **Public Route Table (`courshare-public-rt`):**
   - Associations: Bound to `courshare-public-subnet-1` and `2`.
   - Routes: `0.0.0.0/0` -> Target: `courshare-igw`.
2. **Private Route Table (`courshare-private-rt`):**
   - Associations: Bound to `courshare-private-subnet-1` and `2`.
   - Routes: `0.0.0.0/0` -> Target: Network Interface (ENI) of the **NAT Instance**.
3. **Database Route Table (`courshare-db-rt`):**
   - Associations: Bound to `courshare-db-subnet-1` and `2`.
   - Routes: Local routes only, no outbound route to ensure complete database isolation.

## Step 6: Initialize Security Groups
We will create the following Security Groups (SGs):
* **ALB SG (`courshare-alb-sg`):** Permits inbound HTTP on port `80` and HTTPS on port `443` from any source (`0.0.0.0/0`).
* **ECS Tasks SG (`courshare-ecs-tasks-sg`):** Permits inbound port traffic for ports `8080`, `8081`, `8082`, `8083`, `8084` restricted only from `courshare-alb-sg`.
* **RDS Database SG (`courshare-rds-sg`):** Permits inbound PostgreSQL port `5432` restricted only from `courshare-ecs-tasks-sg` and the NAT Instance (which acts as a Bastion Host).

<!-- TODO: chèn screenshot - [AWS VPC Console showing the 6 created subnets aligned with their CIDR blocks and Availability Zones] -->

<!-- TODO: chèn screenshot - [AWS VPC Route Table showing route 0.0.0.0/0 directed to the Elastic Network Interface of the NAT Instance] -->

<!-- TODO: chèn screenshot - [Inbound Rules of Security Group courshare-rds-sg allowing port 5432 ingress restricted only from the ECS Tasks SG] -->
