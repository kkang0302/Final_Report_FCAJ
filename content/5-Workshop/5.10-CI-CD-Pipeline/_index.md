---
title : "CI/CD Pipeline"
date : 2024-01-01 
weight : 10
chapter : false
pre : " <b> 5.10. </b> "
---

# Automating Deployments with CI/CD Pipelines via GitHub Actions & Amazon ECR

To optimize operation and deployment updates of the CourShare application, we build a fully automated Continuous Integration and Continuous Deployment (CI/CD) system: Whenever a developer pushes new code changes to the `main` branch on **GitHub**, **GitHub Actions** runs tests, builds the Docker image, pushes it to **Amazon ECR**, and commands **AWS ECS Fargate** to update the service without manual intervention.

## Step 1: Store AWS Credentials as GitHub Secrets
To authorize GitHub Actions runners to connect to your AWS account:
1. Create a dedicated IAM User for CI/CD (e.g., `github-cicd-user`) and attach narrow policies: ECR write access (`ecr:*`) and ECS Service updates (`ecs:UpdateService`, `ecs:RegisterTaskDefinition`).
2. Generate an Access Key ID and Secret Access Key for this user.
3. On your microservice GitHub repository, navigate to **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**.
4. Configure 2 repository secrets:
   - `AWS_ACCESS_KEY_ID`: [IAM User Access Key ID]
   - `AWS_SECRET_ACCESS_KEY`: [IAM User Secret Access Key]
   - `AWS_REGION`: `us-east-1`

## Step 2: Draft the GitHub Actions Workflow Configuration
Create a workflow configuration file at `.github/workflows/deploy.yml` inside each microservice repository (e.g., Identity Service):

```yaml
name: Deploy to Amazon ECS

on:
  push:
    branches:
      - main

env:
  ECR_REPOSITORY: courshare-identity-service
  ECS_SERVICE: identity-service
  ECS_CLUSTER: courshare-cluster
  ECS_TASK_DEFINITION: .aws/task-definition.json
  CONTAINER_NAME: identity-service

jobs:
  deploy:
    name: Build & Deploy
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout Code
      uses: actions/checkout@v3

    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ secrets.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, tag, and push image to Amazon ECR
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

    - name: Fill in the new image ID in the Amazon ECS task definition
      id: task-def
      uses: aws-actions/amazon-ecs-render-task-definition@v1
      with:
        task-definition: ${{ env.ECS_TASK_DEFINITION }}
        container-name: ${{ env.CONTAINER_NAME }}
        image: ${{ steps.build-image.outputs.image }}

    - name: Deploy Amazon ECS task definition
      uses: aws-actions/amazon-ecs-deploy-task-definition@v1
      with:
        task-definition: ${{ steps.task-def.outputs.task-definition }}
        service: ${{ env.ECS_SERVICE }}
        cluster: ${{ env.ECS_CLUSTER }}
        wait-for-service-stability: true
```

## Step 3: Pipeline Execution Flow
1. **Trigger:** A developer pushes code changes to the `main` branch.
2. **Build Stage:** The GitHub runner starts an Ubuntu environment, checkouts the repository, and authenticates to Amazon ECR using the secured IAM secrets.
3. **Build & Tag Image:** Packages the Docker image using the service's `Dockerfile`, tags the image with the Git Commit Hash (`github.sha`) for version tracking, and pushes it to ECR.
4. **Deploy Stage:** The `amazon-ecs-deploy-task-definition` Action registers a new Task Definition version reflecting the ECR image URI, then commands the ECS Service to update.
5. **ECS Rolling Update:** AWS ECS executes a rolling update deployment: launches a new container task running the new image, checks its health status via the ALB, and terminates the old container task. This ensures a seamless update flow with **Zero Downtime**.

<!-- TODO: chèn screenshot - [GitHub Settings Secrets screen showing the configured repository secrets for AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY] -->

<!-- TODO: chèn screenshot - [GitHub Actions console showing the deployment workflow successfully completing all checkout, build, push, and deploy tasks] -->
