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

InputsNameDescriptionTypeDefaultRequiredproject_nameName of the project (used for resource tagging)stringn/aYesvpc_cidrThe CIDR block for the VPCstring10.0.0.0/16Nopublic_subnet_cidrsList of CIDR blocks for public subnetslist(string)["10.0.1.0/24", "10.0.2.0/24"]Noavailability_zonesList of AZs to deploy subnets intolist(string)["us-east-1a", "us-east-1b"]No

Name,Description,Type,Default,Required
project_name,Name of the project (used for resource tagging),string,n/a,Yes
vpc_cidr,The CIDR block for the VPC,string,10.0.0.0/16,No
public_subnet_cidrs,List of CIDR blocks for public subnets,list(string),"[""10.0.1.0/24"", ""10.0.2.0/24""]",No
availability_zones,List of AZs to deploy subnets into,list(string),"[""us-east-1a"", ""us-east-1b""]",No

A great README.md is the "front door" of your module. Since this is a remote module, it’s important to document the inputs and outputs so anyone using it doesn't have to dig through your code to figure out how it works.Copy the content below into a file named README.md in the root of your GitHub repository:Markdown# AWS VPC Terraform Module

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
InputsNameDescriptionTypeDefaultRequiredproject_nameName of the project (used for resource tagging)stringn/aYesvpc_cidrThe CIDR block for the VPCstring10.0.0.0/16Nopublic_subnet_cidrsList of CIDR blocks for public subnetslist(string)["10.0.1.0/24", "10.0.2.0/24"]Noavailability_zonesList of AZs to deploy subnets intolist(string)["us-east-1a", "us-east-1b"]NoOutputsNameDescriptionvpc_idThe ID of the created VPCpublic_subnet_idsList of IDs for the created public subnetsRequirementsNameVersionterraform>= 1.0.0aws provider>= 4.0.0

Conversation with Gemini
Create a remote module for VPC do not hardcode the value give variable.tf then upload to github

Building a reusable VPC module is a classic "pro move" for keeping your infrastructure DRY (Don't Repeat Yourself). Since I can't interact with your local terminal to run git commands directly, I’ll provide the structure and the files you need.

Here is a clean, production-ready VPC module setup.

1. The Module Files
You’ll want to put these in a folder named terraform-aws-vpc.

variables.tf
This file ensures nothing is hardcoded, allowing you to pass different values for Dev, Staging, or Prod.

Terraform
variable "vpc_cidr" {
  description = "The CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "project_name" {
  description = "Name of the project for tagging"
  type        = string
}

variable "public_subnet_cidrs" {
  description = "List of CIDR blocks for public subnets"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24"]
}

variable "availability_zones" {
  description = "List of AZs to deploy subnets into"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}
main.tf
This uses the variables defined above to build the resources.

Terraform
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true

  tags = {
    Name = "${var.project_name}-vpc"
  }
}

resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "${var.project_name}-igw"
  }
}

resource "aws_subnet" "public" {
  count                   = length(var.public_subnet_cidrs)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.project_name}-public-subnet-${count.index + 1}"
  }
}
outputs.tf
Crucial for a remote module so other resources can "see" what was created.

Terraform
output "vpc_id" {
  value = aws_vpc.main.id
}

output "public_subnet_ids" {
  value = aws_subnet.public[*].id
}
2. Uploading to GitHub
To make this a "Remote Module," it needs to live in its own repository. Run these commands in your terminal:

Initialize the local repo:

Bash
git init
git add .
git commit -m "Initial commit: VPC remote module"
Create a Repo on GitHub: Go to GitHub and create a new repository (e.g., terraform-aws-vpc).

Push your code:

Bash
git remote add origin https://github.com/YOUR_USERNAME/terraform-aws-vpc.git
git branch -M main
git push -u origin main
3. How to use it (Remote Reference)
Once uploaded, you can call this module in your other projects like this:

Terraform
module "my_vpc" {
  source       = "github.com/YOUR_USERNAME/terraform-aws-vpc"
  project_name = "awesome-app"
  vpc_cidr     = "172.16.0.0/16"
}
Note: For production, it's best practice to append ?ref=v1.0.0 to the source URL to pin the module to a specific Git tag so updates don't accidentally break your infra.

Would you like me to generate a README.md file for this repository so other team members know how to use it?

i have uploaded to github now while using that module to create an infrastructure what all has to be given in the code

Since you’ve successfully pushed the module to GitHub, your root configuration (where you want to build the infrastructure) needs to do three things: call the source, provide the required variables, and initialize the backend.

Here is what your main.tf in your infrastructure project should look like:

1. The Root Configuration (main.tf)
You need to point to your GitHub URL and fill in the blanks for the variables you defined earlier.

Terraform
module "production_vpc" {
  # 1. The Source: Tells Terraform where to download the code
  source = "github.com/YOUR_GITHUB_USERNAME/terraform-aws-vpc"

  # 2. Input Variables: The "Customization" part
  project_name        = "bolt-app"
  vpc_cidr            = "10.0.0.0/16"
  public_subnet_cidrs = ["10.0.1.0/24", "10.0.2.0/24"]
  availability_zones  = ["us-east-1a", "us-east-1b"]
}
2. Essential Configuration Requirements
When using a remote module, there are a few "hidden" requirements you must include for it to work:

Provider Block: Even though the module creates the VPC, your root code must define the AWS provider and the region.

Terraform Init: Because the module is remote, you must run terraform init. This command downloads the code from GitHub into a hidden .terraform folder in your local directory.

Authentication: Your local machine must have AWS credentials (via aws configure or environment variables) and GitHub access (if the repo is private, you’ll need an SSH key or Personal Access Token).

3. Handling Versioning (Best Practice)
If you update your GitHub code later, you might accidentally break your current infrastructure. To prevent this, use Git Tags.

On GitHub, create a release or tag (e.g., v1.0.0).

Change your source line to target that specific version:

Terraform
source = "github.com/YOUR_USERNAME/terraform-aws-vpc?ref=v1.0.0"
4. Viewing the Results
If you want to use the IDs of the VPC or subnets created by the module (for example, to launch an EC2 instance), you reference them like this:

Terraform
resource "aws_instance" "web_server" {
  ami           = "ami-xyz123"
  instance_type = "t3.micro"
  
  # Accessing the output from your remote module
  subnet_id     = module.production_vpc.public_subnet_ids[0] 
}
Would you like me to show you how to set up a providers.tf file to handle the AWS authentication and region settings for this project?

read me for github

A great README.md is the "front door" of your module. Since this is a remote module, it’s important to document the inputs and outputs so anyone using it doesn't have to dig through your code to figure out how it works.

Copy the content below into a file named README.md in the root of your GitHub repository:

Markdown
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
Inputs
Name	Description	Type	Default	Required
project_name	Name of the project (used for resource tagging)	string	n/a	Yes
vpc_cidr	The CIDR block for the VPC	string	10.0.0.0/16	No
public_subnet_cidrs	List of CIDR blocks for public subnets	list(string)	["10.0.1.0/24", "10.0.2.0/24"]	No
availability_zones	List of AZs to deploy subnets into	list(string)	["us-east-1a", "us-east-1b"]	No
Outputs
Name	Description
vpc_id	The ID of the created VPC
public_subnet_ids	List of IDs for the created public subnets
Requirements
Name	Version
terraform	>= 1.0.0
aws provider	>= 4.0.0
License
MIT


---

### Why this structure works:
* **The Usage Block:** Most users just want to "copy-paste" the example and get moving.
* **The Tables:** These provide a quick reference for developers so they know which variables are mandatory (**Yes**) and which have defaults.
* **Outputs Section:** This tells the user how they can connect this VPC to other modul
