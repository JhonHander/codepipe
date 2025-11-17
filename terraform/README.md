# Terraform Infrastructure for AWS ECS

Simple infrastructure as code for deploying the Node.js application on AWS ECS with Fargate.

## What it creates

- VPC with 2 public subnets in different AZs
- Internet Gateway and Route Tables
- Application Load Balancer
- ECS Cluster (Fargate)
- ECR Repository
- ECS Task Definition and Service
- Security Groups
- IAM Roles
- CloudWatch Log Group

## Prerequisites

- Terraform installed
- AWS CLI configured with credentials
- Docker image pushed to ECR (or update task definition after first apply)

## Usage

### 1. Initialize Terraform
```bash
cd terraform
terraform init
```

### 2. Review the plan
```bash
terraform plan
```

### 3. Apply the infrastructure
```bash
terraform apply
```

### 4. Get outputs
```bash
terraform output
```

The ALB DNS name will be displayed. Use it to access your application.

## Important Notes

- The ECS service will initially fail to start because the ECR repository will be empty
- After creating the infrastructure, push your Docker image to ECR:
  ```bash
  # Get ECR URL from outputs
  ECR_URL=$(terraform output -raw ecr_repository_url)
  
  # Build and push
  docker build -t aws-demo-app ..
  docker tag aws-demo-app:latest $ECR_URL:latest
  
  # Login to ECR
  aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $ECR_URL
  
  # Push
  docker push $ECR_URL:latest
  ```

- Then update the ECS service to pull the new image:
  ```bash
  aws ecs update-service --cluster codepipe-cluster --service codepipe-service --force-new-deployment
  ```

## Clean up

To destroy all resources:
```bash
terraform destroy
```

## Costs

This infrastructure uses:
- Fargate (pay per vCPU and memory)
- Application Load Balancer (~$16/month)
- Minimal data transfer and storage costs

Estimated cost: ~$20-30/month if running continuously.
