# Session 18 Homework: Terraform and Infrastructure as Code

## Task 1: Terraform S3 Demo

Create an S3 bucket with Terraform and document the complete infrastructure-as-code workflow. The configuration must be repeatable, formatted, validated, and safely destroyable.

## Project Structure

```text
terraform-s3-demo/
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
├── terraform.tfvars
└── README.md
```

### File Responsibilities

| File | Purpose |
| --- | --- |
| `provider.tf` | Configure the AWS provider and region. |
| `variables.tf` | Declare configurable inputs such as the AWS region and bucket name. |
| `terraform.tfvars` | Provide local variable values for this demo. Do not store credentials here. |
| `main.tf` | Define the S3 bucket and its required settings. |
| `outputs.tf` | Expose useful values such as the bucket name, ARN, and region. |
| `README.md` | Document setup, execution, verification, and cleanup. |

## Prerequisites

Install:

- Terraform CLI
- AWS CLI
- An AWS account with permission to create and delete S3 buckets

Configure credentials using an approved AWS CLI method, for example:

```bash
aws configure
```

Verify the active identity before running Terraform:

```bash
aws sts get-caller-identity
```

Terraform reads AWS credentials from the standard AWS credential chain. Never commit access keys, secret keys, session tokens, or a real `kubeconfig` to the repository.

## S3 Bucket Requirements

The Terraform configuration should:

- Use the `hashicorp/aws` provider.
- Allow the AWS region to be selected through a variable.
- Use a globally unique bucket name, normally by including a project name and random suffix.
- Enable versioning for the bucket.
- Enable server-side encryption with Amazon S3 managed keys or AWS KMS.
- Block public access unless public access is explicitly required.
- Output the bucket name, ARN, and region.

S3 bucket names are globally unique. A name that already exists anywhere in AWS will fail creation, so choose a unique value in `terraform.tfvars`.

## Complete Terraform Workflow

Run all commands from the project directory:

```bash
cd terraform-s3-demo
```

### 1. Initialize Terraform

```bash
terraform init
```

This downloads the required provider and prepares the working directory. A successful run ends with:

```text
Terraform has been successfully initialized!
```

### 2. Format the Configuration

```bash
terraform fmt
```

To check formatting without changing files:

```bash
terraform fmt -check
```

### 3. Validate the Configuration

```bash
terraform validate
```

Expected result:

```text
Success! The configuration is valid.
```

Validation checks the configuration structure and references. It does not contact AWS or create resources.

### 4. Review the Execution Plan

```bash
terraform plan
```

Review the proposed actions carefully. For a new bucket, the summary should be similar to:

```text
Plan: 1 to add, 0 to change, 0 to destroy.
```

For a reviewed, repeatable apply, save the plan:

```bash
terraform plan -out=tfplan
```

The saved plan can be applied exactly as reviewed:

```bash
terraform apply tfplan
```

### 5. Apply the Configuration

To let Terraform show the plan and request confirmation:

```bash
terraform apply
```

Enter `yes` only after reviewing the changes. A successful creation ends with a summary similar to:

```text
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

If the configuration uses additional S3 resources, the resource count may be greater than one.

### 6. Inspect the State and Resources

List resources tracked by the current state:

```bash
terraform state list
```

Inspect the bucket resource:

```bash
terraform state show aws_s3_bucket.demo
```

The resource address must match the name used in `main.tf`.

Show the complete state-derived representation:

```bash
terraform show
```

### 7. Read Terraform Outputs

Show all outputs:

```bash
terraform output
```

Read one output in a script-friendly form:

```bash
terraform output -raw bucket_name
```

The output should contain the bucket name, ARN, and configured AWS region.

Optionally verify the bucket through the AWS CLI:

```bash
aws s3api head-bucket --bucket "$(terraform output -raw bucket_name)"
```

### 8. Destroy the Demo

Preview deletion first:

```bash
terraform plan -destroy
```

Then remove the resources:

```bash
terraform destroy
```

Enter `yes` after confirming that the selected AWS account and bucket are correct. Expected result:

```text
Destroy complete! Resources: 1 destroyed.
```

Verify that no managed resources remain:

```bash
terraform state list
```

The command should return no resource addresses. Keep the local state files private and do not commit them unless the project explicitly uses a secure remote backend.

## Expected Submission Evidence

Include screenshots or terminal output showing:

1. `aws sts get-caller-identity`
2. Successful `terraform init`
3. Successful `terraform fmt` and `terraform validate`
4. A reviewed `terraform plan`
5. Successful `terraform apply`
6. `terraform show` and `terraform output`
7. The bucket visible through the AWS CLI or AWS console
8. Successful `terraform destroy`

Redact account secrets, access keys, session tokens, and other sensitive values before committing evidence.

## Troubleshooting

### Bucket name already exists

S3 names are global. Change the bucket name to a unique value and run `terraform plan` again.

### Access denied

Confirm the active identity with `aws sts get-caller-identity` and verify that it has permission for the S3 actions used by the configuration.

### Wrong AWS account or region

Check the provider region, `terraform.tfvars`, AWS profile, and environment variables before applying. Do not apply changes until the target account is confirmed.

### State lock or stale state

Do not delete state files blindly. Identify whether another Terraform process is running and resolve the lock using the backend-specific procedure.

## Acceptance Checklist

- [ ] The project contains `main.tf`, `variables.tf`, `outputs.tf`, `provider.tf`, `terraform.tfvars`, and `README.md`.
- [ ] `terraform init` succeeds.
- [ ] `terraform fmt` reports no formatting changes.
- [ ] `terraform validate` succeeds.
- [ ] `terraform plan` is reviewed before apply.
- [ ] `terraform apply` creates the S3 bucket.
- [ ] `terraform show` displays the managed resource.
- [ ] `terraform output` displays the documented outputs.
- [ ] `terraform destroy` removes the demo resources.
- [ ] The complete workflow and evidence are documented in this README.
