
```markdown
# EKS Cluster Setup with Terraform

![AWS EKS Architecture](https://example.com/eks-diagram.png) *Replace with actual architecture diagram*

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Infrastructure Components](#infrastructure-components)
- [File Structure](#file-structure)
- [Configuration](#configuration)
- [IAM & Security](#iam--security)
- [State Management](#state-management)
- [Deployment](#deployment)
- [Cleanup](#cleanup)

---

## Overview
Terraform project to provision an AWS EKS cluster with:
- VPC networking infrastructure
- IAM roles with least-privilege policies
- Worker nodes with CloudWatch logging
- State locking using S3 and DynamoDB

---

## Prerequisites
- AWS account with admin privileges
- Terraform v1.0+
- AWS CLI configured
- kubectl (for future cluster management)

---

## Infrastructure Components

### Core Resources
| Component          | Description                          |
|---------------------|--------------------------------------|
| EKS Cluster        | Kubernetes control plane            |
| Managed Node Group | Worker nodes with auto-scaling      |
| VPC                | Isolated network environment        |
| IAM Roles          | Service accounts with fine-grained permissions |

### Observability
- CloudWatch logging enabled
- Audit trail for IAM policies
- 7-day access history tracking

---

## File Structure
```
├── main.tf           # Primary infrastructure declaration
├── variables.tf      # Input variables
├── outputs.tf        # Output values
├── terraform.tfvars  # Variable definitions
└── backend.tf        # Remote state configuration
```

---

## Configuration

### main.tf (Partial)
```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"
  
  cluster_name    = "prod-eks-cluster"
  cluster_version = "1.27"
  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnets
}
```

### Backend Configuration (backend.tf)
```hcl
terraform {
  backend "s3" {
    bucket         = "eks-terraform-state-83"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    dynamodb_table = "terraform-lock"
    encrypt        = true
  }
}
```

---

## IAM & Security

### Key Policies
```markdown
- **Worker Node Policies**
  - AmazonEKSWorkerNodePolicy
  - AmazonEC2ContainerRegistryReadOnly
  - CloudWatchAgentServerPolicy

- **Access Control**
  - 7-day policy rotation
  - Activity-based policy generation
  - Service account isolation
```

### Policy Generation Flow
1. Monitor CloudTrail events
2. Identify required permissions
3. Generate temporary policies
4. Auto-attach to IAM roles
5. Audit weekly

---

## State Management
- **S3 Bucket**: `eks-terraform-state-83`
- **DynamoDB Table**: `terraform-lock`
- **Locking Mechanism**: 
  - Atomic state operations
  - Conflict prevention
  - Versioned state files

---

## Deployment

### Initial Setup
```bash
git clone https://github.com/your-repo/eks-terraform.git
cd eks-terraform
terraform init
```

### Apply Configuration
```bash
terraform apply -var-file="terraform.tfvars"
```

### Verify Resources
```bash
aws eks update-kubeconfig --name prod-eks-cluster
kubectl get nodes
```

---

## Cleanup
```bash
terraform destroy -auto-approve
aws s3 rb s3://eks-terraform-state-83 --force
aws dynamodb delete-table --table-name terraform-lock
```

---

## Important Notes
1. Replace placeholder values in `terraform.tfvars`:
   ```hcl
   aws_region  = "us-west-2"
   cluster_name = "prod-eks-cluster"
   ```
2. Ensure proper IAM permissions before execution
3. Monitor CloudWatch logs during provisioning
4. Store state files securely

*For detailed policy documentation, refer to [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)*
```

