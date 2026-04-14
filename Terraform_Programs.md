# 🏗️ Terraform Programs — AWS Infrastructure with IaC

> **Infrastructure as Code (IaC)** using [Terraform](https://www.terraform.io/) to provision and manage AWS resources.  
> These programs are designed for **beginners to intermediate** learners stepping into the world of cloud automation.

---

## 📋 Table of Contents

1. [Program 1 — Launch Your First EC2 Instance](#program-1--launch-your-first-ec2-instance)
2. [Program 2 — Launch Multiple EC2 Instances Using Count](#program-2--launch-multiple-ec2-instances-using-count)
3. [Program 3 — Secure Provider Config Using AWS CLI Profile](#program-3--secure-provider-config-using-aws-cli-profile)
4. [Program 4 — Dynamic AMI Fetch with Variables & Outputs](#program-4--dynamic-ami-fetch-with-variables--outputs)
5. [Program 5 — EC2 with Security Group, Variables & Outputs](#program-5--ec2-with-security-group-variables--outputs)

---

## ⚙️ Prerequisites

Before running any program, ensure the following are in place:

- [Terraform installed](https://developer.hashicorp.com/terraform/install) (v1.0+)
- An **AWS account** with appropriate IAM permissions
- **AWS CLI** installed and configured (`aws configure`)
- Basic knowledge of the terminal / command line

---

## 🔐 A Note on Secrets & Credentials

> ⚠️ **NEVER hardcode your AWS `access_key` or `secret_key` directly in `.tf` files.**  
> This is a major security risk — your credentials can be exposed if pushed to GitHub.

### ✅ Recommended approach:
- Use **AWS CLI profiles** (`aws configure`) — shown in Program 3 onward
- Use **environment variables**:
  ```bash
  export AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY_HERE"
  export AWS_SECRET_ACCESS_KEY="YOUR_SECRET_KEY_HERE"
  ```
- Use **IAM Roles** for EC2 / CI-CD pipelines

---

## 📦 Common Terraform Commands

```bash
terraform init      # Initialize the working directory & download provider plugins
terraform plan      # Preview what Terraform will create/change/destroy
terraform apply     # Apply the changes and provision resources
terraform destroy   # Tear down all resources created by this configuration
```

---

---

## Program 1 — Launch Your First EC2 Instance

### 📁 File: `main.tf`

```hcl
# ---------------------------------------------------------------
# PROVIDER BLOCK
# Tells Terraform which cloud provider to use and how to connect.
# Here we are using AWS as our cloud provider.
# ---------------------------------------------------------------
provider "aws" {

  # The AWS region where your resources will be created.
  # Example values: "us-east-1", "ap-south-1", "eu-west-1"
  region = "ENTER_YOUR_AWS_REGION"       # <-- Replace with your region e.g. "us-east-1"

  # ⚠️ WARNING: Hardcoding credentials here is NOT recommended for production.
  # Use this ONLY for initial learning/testing in a safe environment.
  # Better alternatives are shown in Program 3 onwards.
  access_key = "ENTER_YOUR_AWS_ACCESS_KEY"    # <-- Replace with your IAM Access Key
  secret_key = "ENTER_YOUR_AWS_SECRET_KEY"    # <-- Replace with your IAM Secret Key
}

# ---------------------------------------------------------------
# RESOURCE BLOCK
# Defines what infrastructure to create.
# Syntax: resource "<PROVIDER_RESOURCE_TYPE>" "<LOCAL_NAME>" {}
# "aws_instance" = EC2 instance resource type in AWS
# "MyFirstInstance" = local name used to reference this resource inside Terraform
# ---------------------------------------------------------------
resource "aws_instance" "MyFirstInstance" {

  # AMI = Amazon Machine Image
  # It is a pre-configured OS image used to launch an EC2 instance.
  # You can find AMI IDs in the AWS Console > EC2 > AMIs section.
  ami = "ENTER_YOUR_AMI_ID"                   # <-- Replace with your AMI ID e.g. "ami-0abcdef1234567890"

  # The type of EC2 instance to launch.
  # "t2.micro" is free-tier eligible for new AWS accounts.
  instance_type = "ENTER_INSTANCE_TYPE"       # <-- Replace with e.g. "t2.micro"

  # Tags are key-value pairs to label/identify AWS resources.
  tags = {
    Name = "terraform-EC2"   # This name appears in the AWS Console EC2 dashboard
  }
}
```

---

### 📖 Line-by-Line Explanation

| Line / Block | What it does | Why it's needed |
|---|---|---|
| `provider "aws" {}` | Declares AWS as the cloud provider | Terraform needs to know which cloud platform and how to authenticate |
| `region` | Sets the geographic region | AWS has multiple data centers globally; we must specify one |
| `access_key` / `secret_key` | Credentials to authenticate with AWS | Required for Terraform to interact with your AWS account |
| `resource "aws_instance" "MyFirstInstance" {}` | Declares an EC2 instance resource | This is the actual infrastructure block that Terraform will create |
| `ami` | Amazon Machine Image ID | Defines the OS (e.g., Amazon Linux, Ubuntu) for the EC2 instance |
| `instance_type` | Size/capacity of the instance | Determines CPU, RAM — `t2.micro` is free-tier eligible |
| `tags` | Metadata labels | Helps identify resources in the AWS Console |

### 🔄 How to Run

```bash
terraform init    # Downloads AWS provider plugin
terraform plan    # Shows what will be created
terraform apply   # Creates the EC2 instance
```

---

---

## Program 2 — Launch Multiple EC2 Instances Using Count

### 📁 File: `main.tf`

```hcl
# ---------------------------------------------------------------
# This program demonstrates how to launch MULTIPLE EC2 instances
# using a single resource block with the "count" meta-argument.
# ---------------------------------------------------------------

# PROVIDER BLOCK — connects Terraform to your AWS account
provider "aws" {

  region     = "ENTER_YOUR_AWS_REGION"         # <-- e.g. "us-east-1"
  access_key = "ENTER_YOUR_AWS_ACCESS_KEY"     # <-- Replace with your IAM Access Key
  secret_key = "ENTER_YOUR_AWS_SECRET_KEY"     # <-- Replace with your IAM Secret Key
}

# ---------------------------------------------------------------
# RESOURCE BLOCK with COUNT
# "count = 3" tells Terraform to create 3 identical instances.
# Terraform uses an internal index (0, 1, 2) for each instance.
# ---------------------------------------------------------------
resource "aws_instance" "multiple_aws_instance_creation" {

  # count = how many instances to create
  count = 3

  ami           = "ENTER_YOUR_AMI_ID"          # <-- Replace with your AMI ID
  instance_type = "ENTER_INSTANCE_TYPE"        # <-- e.g. "t2.micro"

  tags = {
    # count.index gives the current index: 0, 1, 2
    # We add +1 so names start from 1 instead of 0
    # Result: "terraform_EC2-1", "terraform_EC2-2", "terraform_EC2-3"
    Name = "terraform_EC2-${count.index + 1}"
  }
}
```

---

### 📖 Line-by-Line Explanation

| Line / Block | What it does | Why it's needed |
|---|---|---|
| `count = 3` | Creates 3 copies of this resource | Avoids repeating the same resource block multiple times |
| `count.index` | Built-in counter (starts at 0) | Used to differentiate each instance by giving a unique name |
| `${count.index + 1}` | String interpolation with math | Makes names human-friendly: 1, 2, 3 instead of 0, 1, 2 |
| `"terraform_EC2-${...}"` | Dynamic name string | Each EC2 instance gets a unique, identifiable name in AWS Console |

### 💡 Key Concept: `count` Meta-Argument
The `count` meta-argument is a powerful Terraform feature that lets you create **multiple instances of the same resource** without duplicating code. It creates an implicit list:
- `aws_instance.multiple_aws_instance_creation[0]`
- `aws_instance.multiple_aws_instance_creation[1]`
- `aws_instance.multiple_aws_instance_creation[2]`

---

---

## Program 3 — Secure Provider Config Using AWS CLI Profile

### 📁 File: `main.tf`

```hcl
# ---------------------------------------------------------------
# SECURE PROVIDER CONFIGURATION
# This program shows how to AVOID hardcoding credentials.
# Instead, we use the "profile" parameter which reads credentials
# from your local AWS CLI configuration (~/.aws/credentials).
# ---------------------------------------------------------------

provider "aws" {

  # The region where resources will be created.
  # Inline comment shows an example value for students.
  region = "ENTER_YOUR_AWS_REGION"   # Example: "us-east-1", "ap-south-1"

  # "default" refers to the AWS CLI profile name configured on your machine.
  # When you run "aws configure", it creates a profile named "default".
  # Terraform will read the access_key and secret_key from there automatically.
  # ✅ This is the SAFE way — credentials are never written in code.
  profile = "default"
}

# ---------------------------------------------------------------
# RESOURCE BLOCK — Create a single EC2 Instance
# ---------------------------------------------------------------
resource "aws_instance" "EC2_creation" {

  ami           = "ENTER_YOUR_AMI_ID"       # <-- Your AMI ID from AWS Console
  instance_type = "ENTER_INSTANCE_TYPE"     # Example: "t2.micro"

  tags = {
    Name = "EC2_Instance"
  }
}
```

---

### 📖 Line-by-Line Explanation

| Line / Block | What it does | Why it's needed |
|---|---|---|
| `profile = "default"` | Reads credentials from AWS CLI config | Avoids embedding sensitive keys in source code |
| No `access_key` / `secret_key` | Intentionally removed | Prevents accidental exposure of credentials |
| `~/.aws/credentials` (external file) | Stores actual keys on local machine | Terraform reads from here silently and securely |

### 🛠️ How to Set Up the AWS CLI Profile

```bash
# Run this once on your machine to set up credentials:
aws configure

# It will ask for:
# AWS Access Key ID:      [Enter your key]
# AWS Secret Access Key:  [Enter your secret]
# Default region name:    us-east-1
# Default output format:  json
```

This stores credentials safely in `~/.aws/credentials` on your local machine — **not** in your Terraform code.

---

---

## Program 4 — Dynamic AMI Fetch with Variables & Outputs

> This program introduces **multi-file Terraform structure**, **variables**, **data sources**, and **outputs** — the professional way to write Terraform.

### 📁 File Structure

```
project/
├── main.tf        # Main infrastructure definition
├── variables.tf   # Input variable declarations
└── outputs.tf     # Output value declarations
```

---

### 📄 `main.tf`

```hcl
# ---------------------------------------------------------------
# PROVIDER — Uses a variable for region instead of hardcoding it.
# "var.aws_region" reads from variables.tf
# ---------------------------------------------------------------
provider "aws" {
  region  = var.aws_region   # Reads from the variable declared in variables.tf
  profile = "default"        # Uses AWS CLI credentials (secure — no hardcoded keys)
}

# ---------------------------------------------------------------
# DATA SOURCE — Fetches the latest Amazon Linux 2023 AMI dynamically.
# A "data" block READS existing information from AWS (doesn't create anything).
# This avoids hardcoding AMI IDs which change per region and over time.
# ---------------------------------------------------------------
data "aws_ami" "Amazon_Linux" {

  most_recent = true         # Always pick the newest matching AMI

  owners = ["amazon"]        # Only fetch AMIs published by Amazon (trusted source)

  # Filters narrow down which AMI to select
  filter {
    name   = "name"                           # Filter by the AMI name field
    values = ["al2023-ami-*-x86_64"]          # Wildcard pattern: Amazon Linux 2023, 64-bit
  }
}

# ---------------------------------------------------------------
# RESOURCE — Creates the EC2 Instance using the dynamically fetched AMI
# ---------------------------------------------------------------
resource "aws_instance" "EC2_Instance_4" {

  ami           = data.aws_ami.Amazon_Linux.id   # Dynamically retrieved AMI ID
  instance_type = var.instance_type              # From variables.tf
  key_name      = var.key_name                   # SSH key pair name from variables.tf

  tags = {
    Name = "EC2_Instance_4"
  }
}
```

---

### 📄 `variables.tf`

```hcl
# ---------------------------------------------------------------
# VARIABLES FILE
# Declares all input variables used in main.tf.
# Variables make the code reusable and configurable.
# ---------------------------------------------------------------

# Variable for the AWS region
variable "aws_region" {
  description = "AWS Region where resources will be created"   # Human-readable description
  type        = string                                          # Data type: text
  default     = "us-east-1"                                    # Default value if not specified
}

# Variable for the EC2 instance type
variable "instance_type" {
  description = "EC2 instance type (e.g., t2.micro)"
  type        = string
  default     = "t2.micro"    # Free-tier eligible default
}

# Variable for the SSH Key Pair name
# ⚠️ No default here — user MUST provide this value
variable "key_name" {
  description = "Name of an existing EC2 Key Pair for SSH access"
  type        = string
  # No default = Terraform will prompt you to enter this when running apply
}
```

---

### 📄 `outputs.tf`

```hcl
# ---------------------------------------------------------------
# OUTPUTS FILE
# Outputs display useful information after Terraform applies.
# Like "return values" from your infrastructure.
# ---------------------------------------------------------------

# Output the public IP address of the EC2 instance
output "instance_public_ip" {
  value = aws_instance.EC2_Instance_4.public_ip
  # After apply, this will print: instance_public_ip = "x.x.x.x"
}

# Output the unique EC2 Instance ID
output "instance_id" {
  value = aws_instance.EC2_Instance_4.id
  # After apply, this will print: instance_id = "i-0abc123..."
}
```

---

### 📖 Line-by-Line Explanation

| Concept | What it does | Why it's used |
|---|---|---|
| `var.aws_region` | References a variable | Avoids hardcoding; makes config flexible and reusable |
| `data "aws_ami" {}` | Data source block | Reads real-time info from AWS without creating resources |
| `most_recent = true` | AMI filter | Always gets the latest version — keeps infra up to date |
| `owners = ["amazon"]` | Restricts AMI source | Only trust officially published Amazon AMIs |
| `filter {}` | Pattern matching | Matches specific AMI name pattern using wildcard `*` |
| `data.aws_ami.Amazon_Linux.id` | Data reference | Injects the dynamically fetched AMI ID into the resource |
| `variable "key_name"` with no default | Required variable | Forces the user to supply the key pair name at runtime |
| `output "instance_public_ip"` | Output value | Displays the EC2's public IP after provisioning |

### 💡 Key Concepts Introduced

- **Data Sources** (`data` block): Query existing AWS info without creating resources
- **Variables** (`variables.tf`): Parameterize your config for reusability
- **Outputs** (`outputs.tf`): Return useful values after `terraform apply`
- **Multi-file structure**: Cleaner and more maintainable than a single large file

---

---

## Program 5 — EC2 with Security Group, Variables & Outputs

> This is the most complete program — it provisions an EC2 instance inside a **Security Group**, fetching the **VPC** and **AMI** dynamically, with full variable and output support.

### 📁 File Structure

```
project/
├── main.tf        # Infrastructure: VPC data, AMI data, Security Group, EC2
├── variables.tf   # All input variable declarations
└── outputs.tf     # Output values
```

---

### 📄 `main.tf`

```hcl
# ---------------------------------------------------------------
# PROVIDER
# ---------------------------------------------------------------
provider "aws" {
  region  = var.aws_region   # Region comes from variables.tf
  profile = "default"        # Uses local AWS CLI profile (no hardcoded secrets)
}

# ---------------------------------------------------------------
# DATA SOURCE — Fetch the Default VPC
# Every AWS account has a default VPC per region.
# We retrieve it dynamically so we don't hardcode VPC IDs.
# ---------------------------------------------------------------
data "aws_vpc" "default" {
  default = true    # Tells Terraform: fetch the VPC marked as "default" in this region
}

# ---------------------------------------------------------------
# DATA SOURCE — Fetch Latest Amazon Linux 2023 AMI
# Same pattern as Program 4 — always gets the most recent AMI.
# ---------------------------------------------------------------
data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]   # Only official Amazon-published AMIs

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]   # Amazon Linux 2023, 64-bit architecture
  }
}

# ---------------------------------------------------------------
# RESOURCE — Create a Security Group
# A Security Group acts as a virtual firewall for the EC2 instance.
# It controls what traffic is allowed IN (ingress) and OUT (egress).
# ---------------------------------------------------------------
resource "aws_security_group" "EC2_Security_Group" {

  name        = var.security_group_name              # Name from variable
  description = "Security group for EC2 instance"
  vpc_id      = data.aws_vpc.default.id              # Attach to the default VPC (fetched above)

  # INGRESS RULE — Incoming traffic
  # Allows traffic on a specific port (e.g., port 22 for SSH, port 80 for HTTP)
  ingress {
    from_port   = var.ingress_port       # Start of port range
    to_port     = var.ingress_port       # End of port range (same = single port)
    protocol    = var.protocol           # "tcp" or "udp"
    cidr_blocks = var.allowed_cidr       # IP ranges allowed to connect (list)
  }

  # EGRESS RULE — Outgoing traffic
  # Allows ALL outbound traffic (standard practice for EC2 instances)
  egress {
    from_port   = 0                      # 0 means all ports
    to_port     = 0                      # 0 means all ports
    protocol    = "-1"                   # "-1" means ALL protocols
    cidr_blocks = ["0.0.0.0/0"]          # Allow to any destination
  }

  tags = {
    Name = var.security_group_name
  }
}

# ---------------------------------------------------------------
# RESOURCE — Create the EC2 Instance
# The instance is placed inside the Security Group defined above.
# ---------------------------------------------------------------
resource "aws_instance" "EC2_Security_Group" {

  ami           = data.aws_ami.amazon_linux.id                    # Dynamic AMI ID
  instance_type = var.instance_type                               # From variables.tf
  key_name      = var.key_name                                    # SSH key pair name

  # Attach the Security Group to this instance
  vpc_security_group_ids = [aws_security_group.EC2_Security_Group.id]

  tags = {
    Name = var.instance_name   # Instance name from variables.tf
  }
}
```

---

### 📄 `variables.tf`

```hcl
# ---------------------------------------------------------------
# VARIABLES FILE
# All variables have NO defaults here — they must be provided.
# You can supply values via:
#   1. CLI prompt (terraform apply)
#   2. A terraform.tfvars file
#   3. -var flag: terraform apply -var="aws_region=us-east-1"
# ---------------------------------------------------------------

variable "aws_region" {
  description = "AWS Region where resources will be created"
  type        = string
  # Example value: "us-east-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  # Example value: "t2.micro"
}

variable "key_name" {
  description = "Name of an existing EC2 Key Pair for SSH access"
  type        = string
  # Example value: "my-keypair"
}

variable "instance_name" {
  description = "Tag name for the EC2 instance"
  type        = string
  # Example value: "MyWebServer"
}

variable "security_group_name" {
  description = "Name for the Security Group"
  type        = string
  # Example value: "web-sg"
}

variable "ingress_port" {
  description = "Port number to allow inbound traffic on (e.g., 22 for SSH, 80 for HTTP)"
  type        = number
  # Example value: 22
}

variable "protocol" {
  description = "Network protocol for ingress rule"
  type        = string
  # Example value: "tcp"
}

variable "allowed_cidr" {
  description = "List of CIDR blocks allowed for inbound access"
  type        = list(string)
  # Example value: ["0.0.0.0/0"] — allows from anywhere (use with caution!)
  # Recommended: ["YOUR_IP_ADDRESS/32"] — restrict to your IP only
}
```

---

### 📄 `outputs.tf`

```hcl
# ---------------------------------------------------------------
# OUTPUTS
# Displays useful resource info after terraform apply completes.
# ---------------------------------------------------------------

# Public IP of the EC2 instance (used to SSH or access the server)
output "instance_public_ip" {
  value = aws_instance.EC2_Security_Group.public_ip
}

# The ID of the Security Group (useful for referencing in other configs)
output "security_group_id" {
  value = aws_security_group.EC2_Security_Group.id
}
```

---

### 📖 Line-by-Line Explanation

| Concept | What it does | Why it's used |
|---|---|---|
| `data "aws_vpc" "default"` | Fetches the account's default VPC | Avoids hardcoding VPC IDs which differ per account/region |
| `data.aws_vpc.default.id` | Reference to fetched VPC ID | Dynamically links the security group to the correct VPC |
| `resource "aws_security_group"` | Creates a virtual firewall | Controls what traffic can reach the EC2 instance |
| `ingress {}` block | Inbound traffic rule | Defines which ports/protocols/IPs are allowed IN |
| `egress {}` block | Outbound traffic rule | Defines what traffic is allowed OUT (usually all) |
| `from_port` / `to_port` | Port range | Can specify single port (both same) or a range |
| `protocol = "-1"` | All protocols | Used in egress to allow all outbound traffic |
| `cidr_blocks = ["0.0.0.0/0"]` | Allow all IPs | `0.0.0.0/0` = any IP; restrict this in production! |
| `vpc_security_group_ids` | Attach SG to EC2 | Links the security group to the instance |
| `list(string)` type variable | List of CIDR blocks | Allows passing multiple IP ranges for access |
| `type = number` | Numeric variable | Used for port numbers (integers, not strings) |

### 💡 Key Concepts Introduced

- **Security Groups**: Virtual firewall rules for EC2 instances
- **Ingress / Egress rules**: Separate control for incoming vs outgoing traffic
- **`list(string)` variable type**: Passing multiple values as a list
- **`type = number`**: Using integer variables for port numbers
- **No defaults in variables**: Forces explicit values — safer and more controlled
- **Dynamic VPC lookup**: No hardcoded VPC IDs — works across accounts

---

---

## 🗂️ Summary Table

| # | Program Name | Key Concepts |
|---|---|---|
| 1 | Launch Your First EC2 Instance | `provider`, `resource`, `ami`, `instance_type`, `tags` |
| 2 | Launch Multiple EC2 Instances Using Count | `count`, `count.index`, string interpolation |
| 3 | Secure Provider Config Using AWS CLI Profile | `profile`, AWS CLI, credential security |
| 4 | Dynamic AMI Fetch with Variables & Outputs | `data` source, `variable`, `output`, multi-file structure |
| 5 | EC2 with Security Group, Variables & Outputs | Security Groups, `ingress`/`egress`, `list(string)`, VPC data source |

---

## 📚 Further Learning

- [Terraform Official Documentation](https://developer.hashicorp.com/terraform/docs)
- [AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Best Practices](https://developer.hashicorp.com/terraform/language/style)
- [AWS Free Tier](https://aws.amazon.com/free/) — Practice without spending money

---

> 💬 *Have questions? Raise an issue or start a discussion in this repository!*
