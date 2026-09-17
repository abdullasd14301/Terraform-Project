# AWS platform infrastructure with Terraform

This project provisions a small AWS platform from Terraform:

- A VPC with public and private subnets, NAT gateway, and DNS support
- A public Network Load Balancer forwarding TCP/80 to an EC2 Nginx target
- An EKS cluster with one managed node group in the private subnets
- An Amazon Linux EC2 instance in a private subnet
- Jenkins and GitHub Actions workflows for format, validate, plan, and apply

## Prerequisites

Install Terraform 1.6+, AWS CLI, and configure AWS credentials with permission to create VPC, EC2, ELB, IAM, and EKS resources. EKS and NAT gateways can be expensive; destroy the stack when finished.

```bash
cd Terraform-Assignment1
cp terraform.tfvars.example terraform.tfvars
terraform init
terraform plan -var-file=terraform.tfvars
terraform apply -var-file=terraform.tfvars
terraform output
```

To remove the resources:

```bash
terraform destroy -var-file=terraform.tfvars
```

## GitHub Actions

Add the repository secrets `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY_ID` with an IAM access key and secret key. The workflow maps them to the AWS credentials action, runs plan for pull requests and pushes, and applies only on a push to `main`.

## Jenkins

Create two Jenkins Secret Text credentials with IDs `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY_ID`. The `Jenkinsfile` binds them for Terraform as `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. Configure a multibranch pipeline from this repository. Choose the `TERRAFORM_ACTION` build parameter to run `plan`, `apply`, or `destroy`. The `apply` and `destroy` actions are restricted to `main` and require manual approval.

## Production notes

This is a learning baseline. Before production, add an encrypted remote S3 backend with DynamoDB locking, restrict `ssh_cidr`, use separate state per environment, pin the module lock file, add EKS access entries/RBAC, and add private EKS endpoint controls and CloudWatch logging.
