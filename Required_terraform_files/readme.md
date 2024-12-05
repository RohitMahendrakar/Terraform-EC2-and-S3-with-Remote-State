# Terraform Setup for Remote State with AWS S3 and DynamoDB

This document outlines the steps to set up remote state management using AWS S3 and DynamoDB with Terraform. It covers both the remote and local configuration for Terraform state management and the creation of an EC2 instance.

## Steps

### 1. **Set Up Remote State Backend (AWS S3 and DynamoDB)**

#### 1.1 Create a Remote State Configuration with S3 Bucket and DynamoDB Table

In the `remote_state` directory, create the `main.tf` file with the following contents to set up a backend using AWS S3 and DynamoDB:

```hcl
terraform {
  required_version = ">= 0.12"
}

provider "aws" {}

# Create the S3 bucket for storing Terraform state
data "aws_caller_identity" "current" {}

locals {
  account_id = data.aws_caller_identity.current.account_id
}

resource "aws_s3_bucket" "terraform_state" {
  bucket = "${local.account_id}-terraform-states"
  versioning {
    enabled = true
  }

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}

# Create the DynamoDB table for state locking
resource "aws_dynamodb_table" "terraform_lock" {
  name         = "terraform-lock"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

#### 1.2 Create Output Variables for S3 and DynamoDB Resources

Create an `output.tf` file in the `remote_state` directory to output the values of the S3 bucket and DynamoDB table:

```hcl
output "s3_bucket_name" {
  value       = aws_s3_bucket.terraform_state.id
  description = "The NAME of the S3 bucket"
}

output "s3_bucket_arn" {
  value       = aws_s3_bucket.terraform_state.arn
  description = "The ARN of the S3 bucket"
}

output "s3_bucket_region" {
  value       = aws_s3_bucket.terraform_state.region
  description = "The REGION of the S3 bucket"
}

output "dynamodb_table_name" {
  value       = aws_dynamodb_table.terraform_lock.name
  description = "The NAME of the DynamoDB table"
}

output "dynamodb_table_arn" {
  value       = aws_dynamodb_table.terraform_lock.arn
  description = "The ARN of the DynamoDB table"
}
```

#### 1.3 Initialize Terraform

Run the following command to initialize the configuration and create the remote backend:

```bash
terraform init
```

This will create the S3 bucket and DynamoDB table.

### 2. **Set Up Local Configuration for EC2 Instance**

#### 2.1 Create `main.tf` for EC2 Instance in `local_state`

In the `local_state` folder, create a `main.tf` file to define an EC2 instance:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"

  backend "s3" {
    bucket         = "790820559498-terraform-states"
    key            = "terraform.tfstate"  # The file to store the state in S3
    region         = "us-west-2"
    dynamodb_table = "terraform-lock"
    encrypt        = true
  }
}

provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "app_server" {
  ami           = "ami-830c94e3"  # Change this to the desired AMI ID
  instance_type = "t2.micro"

  tags = {
    Name = "Terraform_Demo"
  }
}
```

### 3. **Apply Configuration to Create EC2 Instance**

#### 3.1 Initialize Terraform for Local State

Run the following command to initialize Terraform for the local state:

```bash
terraform init
```

#### 3.2 Apply Terraform to Create EC2 Instance

Run the following command to apply the configuration and create the EC2 instance:

```bash
terraform apply
```

### 4. **Updating State File Backend (S3 and DynamoDB)**

#### 4.1 Modify `main.tf` for Remote State Backend in Local State

Modify the `main.tf` file in the `local_state` directory to point to the remote state backend in the S3 bucket and DynamoDB table:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"

  backend "s3" {
    bucket         = "790820559498-terraform-states"
    key            = "terraform.tfstate"  # The file to store the state in S3
    region         = "us-west-2"
    dynamodb_table = "terraform-lock"
    encrypt        = true
  }
}

provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "app_server" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"

  tags = {
    Name = "Terraform_Demo"
  }
}
```

### 5. **Troubleshooting**

If you encounter errors like `Reference to undeclared resource`, ensure the correct `aws_caller_identity` data source is referenced in your `main.tf` file for remote state configuration.

### 6. **Final Verification**

To verify everything is set up correctly, use the following Terraform commands:

```bash
terraform plan
terraform apply
```

Check the output of `terraform output` to confirm the resources have been created and are working as expected.

---

This guide outlines the steps for configuring Terraform to use AWS S3 and DynamoDB for remote state management, creating an EC2 instance, and handling state management with a backend.
```

This `.md` file summarizes the steps you followed, providing clarity on the configurations and commands executed. Let me know if you'd like to add or modify any part of the documentation!
