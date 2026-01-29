# AWS VPC Terraform Module

This is a reusable Terraform module that deploys a standardized VPC architecture on AWS. It includes a VPC, an Internet Gateway, and configurable public subnets across multiple Availability Zones.

## Features
* **Dynamic Subnets:** Create as many public subnets as you need by passing a list of CIDRs.
* **No Hardcoding:** All values (CIDR, Tags, AZs) are passed as variables.
* **Remote Ready:** Designed to be called from any Terraform root configuration.

---

## Usage

To use this module in your infrastructure, add the following block to your `main.tf`:

```hcl
module "vpc" {
  source              = "[github.com/YOUR_GITHUB_USERNAME/terraform-aws-vpc](https://github.com/YOUR_GITHUB_USERNAME/terraform-aws-vpc)"
  project_name        = "my-awesome-project"
  vpc_cidr            = "10.0.0.0/16"
  public_subnet_cidrs = ["10.0.1.0/24", "10.0.2.0/24"]
  availability_zones  = ["us-east-1a", "us-east-1b"]
}

* **No Hardcoding:** All values (CIDR, Tags, AZs) are passed as variables.
* **Remote Ready:** Designed to be called from any Terraform root configuration.
