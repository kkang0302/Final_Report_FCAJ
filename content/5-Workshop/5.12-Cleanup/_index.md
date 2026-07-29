---
title : "Clean up"
date : 2024-01-01 
weight : 12
chapter : false
pre : " <b> 5.12. </b> "
---

# Guide to Resource Cleanup to Avoid Unwanted Charges

After completing the tests and demonstrating the CourShare project, destroying all provisioned AWS resources is critical to prevent recurring monthly billing charges on your account. Execute the cleanup steps in the following order to ensure no billable components are left orphaned.

## Recommended Cleanup Order

### 1. Destroy ECS Services & Tasks
* Navigate to the **ECS Console** -> Select the `courshare-cluster` cluster -> Open the active services.
* Click **Update Service** -> Set the **Desired tasks** count to `0`.
* Once the active containers have terminated completely, select the services and click **Delete**.
* Finally, delete the **ECS Cluster** `courshare-cluster`.

### 2. Delete the Application Load Balancer
* Go to the **EC2 Console** -> **Load Balancers**.
* Select `courshare-alb` -> Click **Actions** -> **Delete**.
* Next, navigate to **Target Groups**, select the associated Target Groups, and click **Delete**.

### 3. Terminate the RDS Database Instance
* Navigate to the **RDS Console** -> **Databases**.
* Select the database instance `courshare-rds` -> **Actions** -> **Delete**.
* **Note:** Uncheck the "Create final snapshot" option, check the confirmation acknowledging deletion, and execute.

### 4. Terminate the NAT Instance (EC2)
* Go to the **EC2 Console** -> **Instances**.
* Select `courshare-nat-instance` -> **Instance state** -> **Terminate instance**.

### 5. Remove S3 Buckets & CloudFront Distributions
* Go to the **CloudFront Console**, select your distribution, and click **Disable**. Once disabled, click **Delete**.
* Navigate to the **S3 Console**, select `courshare-frontend-web` and `courshare-media-bucket` -> Click **Empty** to wipe the stored objects first, then click **Delete** to remove the buckets.

### 6. Delete Auxiliary Resources
* **Cognito User Pool:** Go to Cognito Console -> Select `courshare-user-pool` -> Click **Delete**.
* **Amazon SQS:** Go to SQS Console -> Select the queues -> Click **Delete**.
* **Amazon ECR:** Go to ECR Console -> Select the repositories -> Click **Delete**.
* **Cloud Map:** Go to Cloud Map Console -> Select Namespace `courshare.local` -> Click **Delete**.
* **VPC:** Lastly, navigate to the VPC Console -> Select `courshare-vpc` -> Click **Delete VPC**. (This will automatically tear down all subnets, route tables, and the Internet Gateway).

<!-- TODO: chèn screenshot - [AWS VPC Console confirming successful deletion of the VPC and all nested components] -->
