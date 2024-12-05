# Terraform EC2 and S3 with Remote State

## Overview
This repository provides a **Terraform configuration** to automate the provisioning of an EC2 instance with AWS, configure remote state management using **Amazon S3** and **DynamoDB**, and create reusable modules for easier management. The aim is to follow best practices for state management, authentication, and environment consistency.

## Prerequisites

Before you begin, ensure the following tools are installed and configured:

1. **AWS CLI**:
   - Install AWS CLI (version 2 recommended).
   - Configure your AWS credentials using the command:
     ```bash
     aws configure
     ```
   - Ensure your AWS IAM user has the necessary permissions to create EC2 instances, S3 buckets, and DynamoDB tables.

2. **Terraform**:
   - Install [Terraform](https://www.terraform.io/downloads.html).
   - Ensure Terraform is installed by running:
     ```bash
     terraform -version
     ```

## Setup

### Clone the Repository
Clone this repository to your local machine to begin:

```bash
git clone https://github.com/username/terraform-ec2-s3-remote-state.git
cd terraform-ec2-s3-remote-state
```

### Initialize Terraform

Terraform uses an initialization process to download the necessary provider plugins and set up the working environment. To initialize, run:

```bash
terraform init
```

This command will download the required AWS provider and configure the backend for state management using S3 and DynamoDB.

### Variables Configuration

In the repository, the following variables are defined:

- **ami**: The Amazon Machine Image ID for the EC2 instance.
- **instance_type**: The EC2 instance type (e.g., `t2.micro`).

You can adjust these variables by editing `input.tf` or passing them during the `terraform apply` command.

### Terraform Commands

1. **Check Plan**:
   To see the execution plan (what resources will be created or modified), run:

   ```bash
   terraform plan
   ```

2. **Apply Changes**:
   To apply the changes defined in the plan and create the infrastructure, use:

   ```bash
   terraform apply
   ```

   This will prompt you to confirm the creation of resources. Type `yes` to proceed.

3. **Destroy Resources**:
   If you want to clean up and remove all the resources created by Terraform, use:

   ```bash
   terraform destroy
   ```

   This command will ask for confirmation before destroying the resources.

### Remote State Configuration

#### Why Use Remote State?

Remote state enables:

- **Collaboration**: Multiple users or processes can work with the same infrastructure without overwriting state files.
- **State Locking**: Prevents concurrent updates to infrastructure and avoids conflicts.
- **Centralized Management**: Keeps the state in a secure, centralized location (S3).

#### Setting Up Remote State with S3 and DynamoDB

To ensure consistent state management, Terraform uses S3 for state storage and DynamoDB for locking.

1. **Create an S3 Bucket** to store the Terraform state file:
   - You can create it via the AWS Console or Terraform itself.

2. **Create a DynamoDB Table** to manage locks:
   - This table will prevent simultaneous operations that could lead to state conflicts.

3. **Configure Backend in Terraform**:
   
In `main.tf`, configure the backend for remote state:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "path/to/my/state.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock-table"
  }
}
```

- `bucket`: The name of your S3 bucket where the state will be stored.
- `key`: The path to the state file within the bucket.
- `dynamodb_table`: The name of the DynamoDB table used for state locking.
- `region`: The AWS region where your S3 bucket and DynamoDB table reside.

### Terraform State File Management

- The Terraform **state file** keeps track of the resources you create and manage.
- **DO NOT commit the `terraform.tfstate` file** to version control.
- **Use S3 for centralized storage** and DynamoDB for locking when working in teams.

#### Security Best Practices:
- Ensure that your state files are encrypted using the `encrypt = true` option in the backend configuration.
- Store sensitive values in **Secrets Manager** or environment variables, not directly in the Terraform configuration files.

## Reusable Modules

Modules are a great way to reuse code and keep your Terraform configuration clean and DRY (Don't Repeat Yourself). Below is an example of how to modularize the EC2 instance configuration:

### EC2 Module
Create a directory `modules/ec2-instance` and define the EC2 resource in `main.tf` within that directory:

```hcl
resource "aws_instance" "example" {
  ami           = var.ami
  instance_type = var.instance_type
}
```

In your root Terraform configuration, you can use the module like so:

```hcl
module "ec2_instance" {
  source         = "./modules/ec2-instance"
  ami            = var.ami
  instance_type  = var.instance_type
}
```

### Benefits of Modules:
- Encapsulates functionality, making it reusable.
- Keeps your main Terraform configuration clean and easy to manage.
- Easier to manage dependencies and variables across different environments.

## Output Configuration

Once resources are created, you may want to output values such as the EC2 instance’s public IP.

Define outputs in `output.tf`:

```hcl
output "instance_public_ip" {
  value = aws_instance.example.public_ip
}
```

Outputs make it easier to share useful information, especially in automated pipelines or team environments.

## Git Best Practices

- **Add `terraform.tfstate` and `*.tfstate.backup` to `.gitignore`** to ensure that state files are not accidentally committed to version control.
  
Example `.gitignore`:

```bash
*.tfstate
*.tfstate.backup
```

This ensures sensitive data is kept out of your version control system.

## Cleanup

To destroy the infrastructure and delete all resources, run:

```bash
terraform destroy
```

Ensure that you have reviewed the planned changes and confirm the deletion.

## CI/CD Integration

### Jenkins Pipeline

You can easily integrate this Terraform setup with Jenkins pipelines to automate infrastructure provisioning. For this, you’ll need:

1. **AWS CLI Configuration**: Make sure Jenkins has access to the AWS account.
2. **Environment Variables**: Store AWS credentials and other sensitive information as Jenkins environment variables.

The output generated by `output.tf` can be used in the pipeline to capture the EC2 instance's public IP or any other required information.

## Contributing

Feel free to open issues or submit pull requests to improve this repository. If you're adding new features or fixing bugs, please ensure that:
- You add tests where necessary.
- Documentation is updated to reflect the changes.
  
## License

This repository is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgements

- Terraform documentation: https://www.terraform.io/docs
- AWS SDKs and CLI documentation: https://docs.aws.amazon.com/
- Terraform AWS Provider: https://registry.terraform.io/providers/hashicorp/aws/latest/docs
```

---

This enhanced `README.md` now includes:

1. **Detailed Instructions**: Clear guidance on every step, including setup, initialization, running commands, and CI/CD integration.
2. **Security Practices**: Best practices on handling Terraform state files, including using remote backends and avoiding storing sensitive information.
3. **Reusable Modules**: Explanation of how to structure and use Terraform modules for better maintainability and scalability.
4. **CI/CD Pipeline Integration**: Tips on how to integrate Terraform with Jenkins.
5. **Contributing Guidelines**: Encouragement for others to contribute to the project, including guidelines for pull requests and issue submissions.

This structure makes it much easier for others to use and contribute to the repository.
