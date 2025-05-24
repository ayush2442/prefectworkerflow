
# Prefect Worker Deployment on Amazon ECS Fargate (via AWS CloudFormation)

This project demonstrates the deployment of a Prefect worker (`dev-worker`) on Amazon ECS using the Fargate launch type, defined entirely through Infrastructure as Code (IaC) with AWS CloudFormation.

---

## Tooling & Stack

- **IaC Tool**: AWS CloudFormation (YAML)
- **Container Orchestration**: Amazon ECS (Fargate)
- **Orchestrator**: Prefect Cloud
- **Networking**: Custom VPC, Public & Private Subnets
- **Secrets Management**: AWS Secrets Manager

---

## Folder Structure

```
.
├── vpc.yaml              # CloudFormation for VPC, subnets, NAT, etc.
├── ecs_prefect.yaml      # ECS cluster, task definition, IAM, service
├── README.md             # Documentation (this file)
```

---

## Setup Instructions

### 1. Prerequisites

- AWS account with permission to create VPC, IAM roles, ECS resources.
- AWS CLI configured.
- Prefect Cloud account with:
  - Account ID
  - Workspace ID
  - API key

### 2. Store Prefect API Key in Secrets Manager

```bash
aws secretsmanager create-secret --name prefect-api-key --secret-string "<your-prefect-api-key>"
```

Save the **Secret ARN** for use in the next steps.

---

## Deployment Steps

### 1. Deploy VPC and Networking

```bash
aws cloudformation deploy --template-file vpc.yaml --stack-name prefect-vpc-stack --capabilities CAPABILITY_NAMED_IAM
```

Note down:
- Private subnet IDs
- Security group ID

### 2. Deploy ECS Cluster, IAM Roles, and Prefect Worker

Update `ecs_prefect.yaml` parameters:
- Replace `SubnetIds`, `SecurityGroupIds`, and `PrefectAPISecretArn`
- Set your `PrefectAccountId`, `WorkspaceId`, and `WorkPoolName`

Then deploy:

```bash
aws cloudformation deploy --template-file ecs_prefect.yaml --stack-name prefect-ecs-stack --capabilities CAPABILITY_NAMED_IAM
```

---

## Verification Steps

### In AWS Console:

- Go to **ECS > Clusters** → Verify `prefect-cluster` is running.
- Go to **ECS > Tasks** → Check that a task is RUNNING and not stopped.
- Go to **CloudWatch Logs** → Inspect worker logs if task stops unexpectedly.

### In Prefect Cloud:

1. Go to [app.prefect.cloud](https://app.prefect.cloud)
2. Navigate to your **Workspace > Work Pools**
3. If not already created, manually create a **Work Pool**:
   - Name: `ecs-work-pool`
   - Type: `Amazon ECS` (only available on paid plans)
4. The worker `dev-worker` should appear as **Online** under that pool.

---

## Cleanup Instructions

To remove all deployed resources:

```bash
aws cloudformation delete-stack --stack-name prefect-ecs-stack
aws cloudformation delete-stack --stack-name prefect-vpc-stack
```

Also manually delete the secret from AWS Secrets Manager:

```bash
aws secretsmanager delete-secret --secret-id prefect-api-key --force-delete-without-recovery
```

---

## Author

Ayush Pandey
