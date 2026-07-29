---
title : "RDS Database Layer"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

# Deploying the Amazon RDS (PostgreSQL) Database Layer

Each microservice in the CourShare system requires its own database schema. To keep testing costs optimal (avoiding launching 5 separate RDS instances which would cost hundreds of dollars monthly), we design a **Multi-schema** architecture running on a single, shared **Amazon RDS PostgreSQL** instance, securely hidden within the isolated Database Subnets.

## Step 1: Create a DB Subnet Group
To support failover configurations, AWS RDS requires database instances to associate with at least 2 Subnets in different Availability Zones.
1. Navigate to the **RDS Console** -> **Subnet groups** -> **Create DB Subnet Group**.
2. Name: `courshare-db-subnet-group`.
3. VPC: Select `courshare-vpc`.
4. Add subnets: Choose AZs `us-east-1a`, `us-east-1b` and select the correct Database Subnets: `courshare-db-subnet-1` (`10.0.20.0/24`) and `courshare-db-subnet-2` (`10.0.21.0/24`).
5. Click **Create**.

## Step 2: Initialize the RDS Database Instance
1. Go to the **RDS Console** -> **Databases** -> **Create database**.
2. Choose database creation method: **Standard create**.
3. Engine options: Select **PostgreSQL**.
4. Templates: Select **Free Tier** (or Dev/Test).
5. Settings:
   - DB instance identifier: `courshare-rds`.
   - Master username: `courshare`.
   - Master password: `courshare-secure-password`.
6. Instance configuration:
   - DB instance class: Choose **db.t4g.micro** (highly cost-efficient, optimal for dev/test environments).
7. Connectivity:
   - Virtual Private Cloud (VPC): Select `courshare-vpc`.
   - DB Subnet Group: Select `courshare-db-subnet-group`.
   - Public access: **No** (Strict security, denying direct access from the public internet).
   - VPC Security Group: Choose **Choose existing** -> Remove default SG and select `courshare-rds-sg`.
8. Additional configuration:
   - Initial database name: `courshare_db`.
9. Click **Create database**.

## Step 3: Initialize the 5 Database Schemas via Bastion Host
Since the RDS instance resides in the Database Subnet without a public IP, we SSH into the **NAT Instance** (situated in the Public Subnet) acting as a Bastion Host to manage our database:

1. SSH into the NAT instance from your local machine.
2. Install the PostgreSQL client on the NAT instance:
   ```bash
   sudo dnf install postgresql15 -y
   ```
3. Connect to the RDS instance using the allocated DB Endpoint:
   ```bash
   psql -h courshare-rds.xxxxxx.us-east-1.rds.amazonaws.com -U courshare -d courshare_db
   ```
4. Execute SQL commands to create the 5 isolated schemas for the microservices:
   ```sql
   CREATE SCHEMA identity_db;
   CREATE SCHEMA course_db;
   CREATE SCHEMA enrollment_db;
   CREATE SCHEMA learning_db;
   CREATE SCHEMA payment_db;
   ```

## Step 4: Connect ECS Tasks to RDS
Update database connection environment variables in the ECS Task Definition for each service:

* **Identity Service (Java Spring Boot):**
  - Environment Variable: `SPRING_DATASOURCE_URL = jdbc:postgresql://courshare-rds.xxxxxx.us-east-1.rds.amazonaws.com:5432/courshare_db?currentSchema=identity_db`
* **Course Service (Java Spring Boot):**
  - Environment Variable: `SPRING_DATASOURCE_URL = jdbc:postgresql://courshare-rds.xxxxxx.us-east-1.rds.amazonaws.com:5432/courshare_db?currentSchema=course_db`
* **Enrollment Service (Node.js/Express):**
  - Environment Variable: `DATABASE_URL = postgresql://courshare:courshare-secure-password@courshare-rds.xxxxxx.us-east-1.rds.amazonaws.com:5432/courshare_db?schema=enrollment_db`
* **Learning Service (Node.js/Express):**
  - Environment Variable: `DATABASE_URL = postgresql://courshare:courshare-secure-password@courshare-rds.xxxxxx.us-east-1.rds.amazonaws.com:5432/courshare_db?schema=learning_db`
* **Payment Service (Node.js/Express):**
  - Environment Variable: `DATABASE_URL = postgresql://courshare:courshare-secure-password@courshare-rds.xxxxxx.us-east-1.rds.amazonaws.com:5432/courshare_db?schema=payment_db`

*(Upon redeployment, the ECS Tasks resolve the database endpoint via internal VPC routing, automatically executing schema-level migration scripts to initialize their respective tables).*

<!-- TODO: chèn screenshot - [Amazon RDS Console showing the database instance courshare-rds in Available status showing the database connectivity endpoint] -->

<!-- TODO: chèn screenshot - [DBeaver client connected to the RDS database showing schemas and tables like users, courses, enrollments, and payments generated inside their independent schemas] -->
