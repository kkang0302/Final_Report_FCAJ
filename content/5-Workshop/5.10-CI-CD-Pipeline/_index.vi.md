---
title : "Pipeline CI/CD"
date : 2024-01-01 
weight : 10
chapter : false
pre : " <b> 5.10. </b> "
---

# Tự động hóa Triển khai với Pipeline CI/CD qua GitHub Actions & Amazon ECR

Để tối ưu hóa quá trình vận hành và cập nhật sản phẩm CourShare, chúng ta xây dựng một hệ thống tích hợp và triển khai liên tục (CI/CD) tự động hóa hoàn toàn: Mỗi khi lập trình viên đẩy mã nguồn mới lên nhánh `main` của **GitHub**, **GitHub Actions** sẽ tự động chạy tests, build Docker image, push lên **Amazon ECR** và ra lệnh cho **AWS ECS Fargate** cập nhật dịch vụ mà không cần thao tác thủ công.

## Bước 1: Đăng ký AWS Credentials làm GitHub Secrets
Để GitHub Actions có quyền kết nối tới tài khoản AWS của bạn:
1. Tạo một IAM User chuyên dụng cho CI/CD (ví dụ `github-cicd-user`) và cấp quyền hạn hẹp: truy cập ECR (`ecr:*`) và cập nhật ECS Service (`ecs:UpdateService`, `ecs:RegisterTaskDefinition`).
2. Sinh Access Key ID và Secret Access Key cho user này.
3. Trên repository GitHub của từng microservice, truy cập **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**.
4. Khởi tạo 2 secrets:
   - `AWS_ACCESS_KEY_ID`: [Access Key ID của IAM User]
   - `AWS_SECRET_ACCESS_KEY`: [Secret Access Key của IAM User]
   - `AWS_REGION`: `us-east-1`

## Bước 2: Soạn thảo cấu hình Workflow GitHub Actions
Tạo tệp tin cấu hình tại đường dẫn `.github/workflows/deploy.yml` trong repository của microservice (ví dụ: Identity Service):

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

## Bước 3: Cơ chế hoạt động của Pipeline
1. **Trigger:** Lập trình viên push code lên nhánh `main`.
2. **Build Stage:** GitHub runner khởi chạy một container Ubuntu, tải source code, đăng nhập vào Amazon ECR bằng IAM credentials bảo mật.
3. **Build & Tag Image:** Docker build image dựa trên `Dockerfile` của service, tag image bằng Git Commit Hash (`github.sha`) để dễ quản lý phiên bản và push lên ECR.
4. **Deploy Stage:** Action `amazon-ecs-deploy-task-definition` đăng ký một phiên bản Task Definition mới chứa link image vừa push, sau đó cập nhật ECS Service.
5. **ECS Rolling Update:** AWS ECS thực hiện quá trình cập nhật cuốn chiếu (Rolling Update): Khởi chạy task mới chứa image mới; sau khi task mới vượt qua health check của ALB thành công, ECS mới thực hiện shutdown task cũ. Luồng này giúp hệ thống cập nhật mượt mà, đảm bảo **Zero Downtime**.

<!-- TODO: chèn screenshot - [Màn hình GitHub Settings Secrets hiển thị các biến AWS_ACCESS_KEY_ID và AWS_SECRET_ACCESS_KEY đã được khai báo bảo mật] -->

<!-- TODO: chèn screenshot - [Màn hình GitHub Actions chạy thành công toàn bộ các step từ checkout, login ECR đến update ECS Service thành công] -->
