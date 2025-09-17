# 🚀 3-Tier Infrastructure Deployment Using Terraform Modules

## Introduction
This project demonstrates the creation and deployment of a **3-tier architecture** on **Amazon Web Services (AWS)** using **Terraform modules**. It follows industry best practices for modular, scalable, and secure infrastructure as code (IaC).

---

## 🧾 Project Overview

This 3-tier architecture follows a widely adopted software design pattern that organizes applications into **Web**, **Application**, and **Database** tiers. Each tier is:

- **Isolated** in its own subnet
- **Secured** using Terraform-managed security groups
- **Provisioned** using reusable Terraform modules

---

## 🏗️ Architecture Overview

### 1️⃣ Web Tier
- Deployed in public subnets
- EC2 instances with a web server (Apache/Nginx)
- Accessible via HTTP/HTTPS and SSH

### 2️⃣ Application Tier
- Deployed in private subnets
- EC2 instances running backend services (e.g., Python/Node.js)
- Communicates only with Web Tier and DB Tier
- No direct internet access

### 3️⃣ Database Tier
- Deployed in private subnets
- Amazon RDS MySQL
- Accessible only from Application Tier

---

## 🌐 Network Design

- Custom **VPC**
- **Public Subnets** (Web Tier)
- **Private Subnets** (App and DB Tiers)
- **Internet Gateway** for external traffic
- **NAT Gateway** for private subnet egress
- Route tables with proper subnet associations

---

## 🔐 Security Groups

| Layer         | Inbound Access From               |
|---------------|-----------------------------------|
| Web Tier      | HTTP/HTTPS/SSH from internet      |
| App Tier      | Custom port (e.g., 8080) from Web |
| Database Tier | MySQL (3306) from App Tier        |

---

## 📁 Project Structure

```

3-tier-terraform/
├── main.tf               # Root Terraform file invoking modules
├── variables.tf          # Input variable definitions
├── outputs.tf            # Output variable definitions
├── terraform.tfvars      # Actual variable values
│
├── modules/
│   ├── vpc/              # VPC, Subnets, NAT, IGW
│   ├── web/              # EC2 instance setup for Web Tier
│   ├── app/              # EC2 instance setup for App Tier
│   ├── db/               # RDS MySQL instance setup
│   └── security/         # Security Groups for all tiers

````

---

## 🛠 Setup Instructions (Using VS Code)

### 1. Create Project Folder
- Navigate to Desktop and create a folder named: `3-tier-terraform`.

### 2. Open in VS Code
- Launch Visual Studio Code.
- Go to **File > Open Folder**, and open `3-tier-terraform`.

### 3. Create Root Files
Create the following files:
- `main.tf`
- `variables.tf`
- `outputs.tf`
- `terraform.tfvars`

### 4. Create Module Structure
Inside `3-tier-terraform`, create a folder named `modules`.
Inside `modules`, create these directories:
- `vpc`
- `web`
- `app`
- `db`
- `security`

Each module folder should contain its own:
- `main.tf`
- `variables.tf`
- `outputs.tf`

### 5. Initialize Terraform
```bash
terraform init
```
---
## 🚀 Step-by-Step Code Setup

This guide walks you through setting up and writing Terraform code for a 3-tier AWS infrastructure using modular design.

---

## 📂 Step 1: Open Root Directory

1. Open the folder `3-tier-terraform` in **Visual Studio Code**.
2. You should see the following structure:


```

3-tier-terraform/
├── main.tf               # Root Terraform file invoking modules
├── variables.tf          # Input variable definitions
├── outputs.tf            # Output variable definitions
├── terraform.tfvars      # Actual variable values
├── backend.tf            # Optional backend config for remote state (S3/DynamoDB)
│
├── modules/
│   ├── vpc/              # VPC, Subnets, NAT, IGW
│   ├── web/              # EC2 instance setup for Web Tier
│   ├── app/              # EC2 instance setup for App Tier
│   ├── db/               # RDS MySQL instance setup
│   └── security/         # Security Groups for all tiers

````
---

## ✍️ Step 2: Add Code to Root Files

### 🔹 main.tf (Root)

Open `main.tf` and add:

```hcl
terraform {
  required_version = "~> 1.1"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
 
# Configure the AWS Provider
provider "aws" {
  region = var.region
}
module "vpc" {
  source    = "./modules/vpc"
  region = var.region
  vpc_cidr  = var.vpc_cidr
}

module "security" {
  source = "./modules/security"
  vpc_id = module.vpc.vpc_id
}

module "web" {
  source             = "./modules/web"
  public_subnets     = module.vpc.public_subnets
  ami_id             = var.web_ami
  instance_type      = var.web_instance_type
  security_group_id  = module.security.web_sg_id
}

module "app" {
  source             = "./modules/app"
  private_subnets    = module.vpc.app_subnets
  ami_id             = var.app_ami
  instance_type      = var.app_instance_type
  security_group_id  = module.security.app_sg_id
}

module "db" {
  source             = "./modules/db"
  db_subnets         = module.vpc.db_subnets
  db_username        = var.db_username
  db_password        = var.db_password
  security_group_id  = module.security.db_sg_id
}
```
### 🔹 variables.tf (Root)

Open `variables.tf` and add:

```hcl
variable "region" {}
variable "vpc_cidr" {}
variable "web_ami" {}
variable "web_instance_type" {}
variable "app_ami" {}
variable "app_instance_type" {}
variable "db_username" {}
variable "db_password" {}

```
### 🔹 outputs.tf (Root)

Open `variables.tf` and add:

```hcl
output "web_instance_ids" {
  value = module.web.instance_ids
}

output "app_instance_ids" {
  value = module.app.instance_ids
}

output "db_endpoint" {
  value = module.db.db_endpoint
}
```
### 🔹 terraform.tfvars (Root)

Open `terraform.tf` and add:

```hcl
vpc_cidr = "10.0.0.0/16"
region = "ap-south-1"
web_ami = "ami-0e35ddab05955cf57"
app_ami = "ami-0e35ddab05955cf57"
web_instance_type = "t2.micro"
app_instance_type = "t2.micro"
db_username = "admin"
db_password = "Pass$123"
```

## 🧱 Step 3: Add Code to Modules

### 🔹 main.tf (module/vpc)

open `modules/vpc/main.tf` and add:

```hcl
  resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  enable_dns_support = true
  enable_dns_hostnames = true
}

resource "aws_subnet" "public" {
  count = 2
  vpc_id = aws_vpc.main.id
  cidr_block = cidrsubnet(var.vpc_cidr, 4, count.index)
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.available.names[count.index]
}

resource "aws_subnet" "app" {
  count = 2
  vpc_id = aws_vpc.main.id
  cidr_block = cidrsubnet(var.vpc_cidr, 4, count.index + 2)
  availability_zone = data.aws_availability_zones.available.names[count.index]
}

resource "aws_subnet" "db" {
  count = 2
  vpc_id = aws_vpc.main.id
  cidr_block = cidrsubnet(var.vpc_cidr, 4, count.index + 4)
  availability_zone = data.aws_availability_zones.available.names[count.index]
}

data "aws_availability_zones" "available" {}

output "vpc_id" {
  value = aws_vpc.main.id
}

output "public_subnets" {
  value = aws_subnet.public[*].id
}

output "app_subnets" {
  value = aws_subnet.app[*].id
}

output "db_subnets" {
  value = aws_subnet.db[*].id
}

```
### 🔹 variables.tf (module/vpc)

open `modules/vpc/variables.tf` and add:

```hcl
variable "vpc_cidr" {}
variable "region" {}
```
### 🔹 main.tf (module/security)

open `modules/security/main.tf` and add:

```hcl
resource "aws_security_group" "web_sg" {
  name = "web-sg"
  description = "Allow HTTP"
  vpc_id = var.vpc_id

  ingress {
    from_port = 80
    to_port   = 80
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port = 0
    to_port   = 0
    protocol  = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "app_sg" {
  name = "app-sg"
  description = "Allow from Web"
  vpc_id = var.vpc_id

  ingress {
    from_port = 8080
    to_port   = 8080
    protocol  = "tcp"
    security_groups = [aws_security_group.web_sg.id]
  }
  egress {
    from_port = 0
    to_port   = 0
    protocol  = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "db_sg" {
  name = "db-sg"
  description = "Allow MySQL from App"
  vpc_id = var.vpc_id

  ingress {
    from_port = 3306
    to_port   = 3306
    protocol  = "tcp"
    security_groups = [aws_security_group.app_sg.id]
  }
  egress {
    from_port = 0
    to_port   = 0
    protocol  = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

output "web_sg_id" {
  value = aws_security_group.web_sg.id
}

output "app_sg_id" {
  value = aws_security_group.app_sg.id
}

output "db_sg_id" {
  value = aws_security_group.db_sg.id
}

```
### 🔹 variables.tf (module/security)

open `modules/security/variables.tf` and add:

```hcl
variable "vpc_id" {}
```
### 🔹 main.tf (module/web)

open `modules/web/main.tf` and add:

```hcl
resource "aws_instance" "web" {
  count = 2
  ami = var.ami_id
  instance_type = var.instance_type
  subnet_id = element(var.public_subnets, count.index)
  vpc_security_group_ids = [var.security_group_id]
  tags = { Name = "web-${count.index+1}" }
}

output "instance_ids" {
  value = aws_instance.web[*].id
}


```
### 🔹 variables.tf (module/web)

open `modules/web/variables.tf` and add:

```hcl
variable "ami_id" {}
variable "instance_type" {}
variable "public_subnets" {}
variable "security_group_id" {}

```
### 🔹 main.tf (module/app)

open `modules/app/main.tf` and add:

```hcl
resource "aws_instance" "app" {
  count = 2
  ami = var.ami_id
  instance_type = var.instance_type
  subnet_id = element(var.private_subnets, count.index)
  vpc_security_group_ids = [var.security_group_id]
  tags = { Name = "app-${count.index+1}" }
}

output "instance_ids" {
  value = aws_instance.app[*].id
}

```
### 🔹 variables.tf (module/app)

open `modules/app/variables.tf` and add:

```hcl
variable "ami_id" {} 
variable "instance_type" {}
variable "private_subnets" {}
variable "security_group_id" {}
```
### 🔹 variables.tf (module/db)

open `modules/db/main.tf` and add:

```hcl
resource "aws_db_subnet_group" "db" {
  name       = "db-subnet-group"
  subnet_ids = var.db_subnets
}

resource "aws_db_instance" "mysql" {
  allocated_storage    = 20
  engine               = "mysql"
  engine_version       = "8.0"
  instance_class       = "db.t3.micro"
  #name                 = "appdb"
  username             = var.db_username
  password             = var.db_password
  skip_final_snapshot  = true
  db_subnet_group_name = aws_db_subnet_group.db.name
  vpc_security_group_ids = [var.security_group_id]
  publicly_accessible = false
}

output "db_endpoint" {
  value = aws_db_instance.mysql.endpoint
}

```
### 🔹 variables.tf (module/db)

open `modules/db/variables.tf` and add:

```hcl
variable "db_subnets" {}
variable "db_username" {}
variable "db_password" {}
variable "security_group_id" {}
  
```

---

## 🛠 Tools Used

| Tool         | Purpose                        |
|--------------|--------------------------------|
| Terraform    | Infrastructure provisioning   |
| AWS EC2      | Compute instances              |
| AWS RDS      | Managed MySQL DB               |
| AWS VPC      | Networking and isolation       |

---

## 🚀 Deployment Steps

```bash
# Clone the repository
git clone https://github.com/Swatiz-cloud/3-tier-terraform-modules.git
cd 3-tier-terraform

# Configure variable values in terraform.tfvars

# Initialize Terraform
terraform init

# Validate configuration
terraform validate

# Preview the changes
terraform plan

# Apply the configuration
terraform apply
````

---

## 📤 Output Values

* Public IP of Web EC2 instance
* Private IPs of App EC2 instance
* RDS MySQL endpoint
* Subnet and VPC IDs

---

## 🛡️ Security Measures

* Public access restricted to Web Tier only
* App and DB tiers in private subnets
* NAT Gateway used for secure outbound access
* Sensitive data stored securely in `.tfvars`

---

## 🧹 Cleanup Instructions

```bash
# Destroy all infrastructure
terraform destroy

# Optional cleanup
rm -rf .terraform terraform.tfstate*
```

Verify AWS Console to ensure all resources are removed.

---

## 📈 Benefits of Modular Terraform

* 🔍 **Clarity** – Separation of concerns improves maintainability
* ♻️ **Reusability** – Modules can be reused in different environments
* 📏 **Consistency** – Predictable and parameterized deployments
* 🚀 **Scalability** – Easy to add/remove components
* 🤖 **Automation** – Full IaC workflow

---

## 🧪 Future Enhancements

* Add ALB for Web Tier
* Enable Auto Scaling Groups
* Use Secrets Manager or SSM for DB credentials
* Enable HTTPS via ACM
* Add CloudWatch for logging and monitoring
* Use remote backend with S3 & DynamoDB

---

## ✅ Conclusion

This project provides a **modular, secure, and scalable** AWS infrastructure using Terraform. It is a strong base for production-ready applications and can easily be extended for high availability, automation, and DevOps pipelines.



