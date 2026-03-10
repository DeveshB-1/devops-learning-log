# Terraform Notes

## Core Concepts

| Term | Description |
|---|---|
| Provider | Plugin to interact with APIs (AWS, Azure, GCP) |
| Resource | Infrastructure object to create/manage |
| Data source | Read existing infrastructure |
| Module | Reusable group of resources |
| State | Map of real infra to config (`.tfstate`) |
| Workspace | Separate state environments |

## Basic Workflow

```bash
terraform init      # download providers
terraform plan      # show what will change
terraform apply     # apply changes
terraform destroy   # delete everything
terraform fmt       # format code
terraform validate  # check syntax
```

## AWS VPC Example

```hcl
# providers.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# variables.tf
variable "aws_region" {
  default = "ap-south-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
}

# main.tf
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-south-1a"
  map_public_ip_on_launch = true
  tags = {
    Name = "${var.environment}-public-subnet"
  }
}

# outputs.tf
output "vpc_id" {
  value = aws_vpc.main.id
}
```

## Remote State (S3 + DynamoDB)

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "ap-south-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"  # for state locking
  }
}
```

## Module Structure

```
modules/
  vpc/
    main.tf
    variables.tf
    outputs.tf
  ec2/
    main.tf
    variables.tf
    outputs.tf

environments/
  dev/
    main.tf     # calls modules
    variables.tf
  prod/
    main.tf
    variables.tf
```

## Key Tips

- Always use remote state in teams
- Use `terraform.tfvars` for environment-specific values
- Never hardcode credentials — use IAM roles or env vars
- Use `count` or `for_each` for multiple similar resources
- `terraform import` to bring existing infra under management
