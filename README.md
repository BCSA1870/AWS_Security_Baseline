# AWS_Security_Baseline
Production-ready Terraform modules for secure AWS environment provisioning implementing CIS AWS Foundations Benchmark security controls. Technologies: Terraform, AWS (IAM, VPC, GuardDuty, SecurityHub, CloudTrail, Config, KMS), Python

# AWS Security Baseline - Terraform Implementation Guide
## Production-Ready Infrastructure as Code with CIS Benchmark Compliance

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Prerequisites](#prerequisites)
3. [Architecture Overview](#architecture-overview)
4. [Step-by-Step Implementation](#step-by-step-implementation)
5. [Terraform Module Structure](#terraform-module-structure)
6. [Python Monitoring & Enforcement](#python-monitoring-enforcement)
7. [Testing & Validation](#testing-validation)
8. [Deployment Guide](#deployment-guide)

---

## Project Overview

This project provides production-ready Terraform modules that implement AWS security best practices aligned with the CIS AWS Foundations Benchmark. It automates the provisioning of a secure AWS environment with hardened configurations, monitoring, and continuous compliance enforcement.

**Key Features:**
- Hardened VPC architecture with security group templates
- IAM baseline with least privilege access controls
- Comprehensive logging and monitoring (CloudTrail, Config, GuardDuty, Security Hub)
- Encrypted S3 configurations with lifecycle policies
- Automated compliance monitoring and remediation
- Multi-account support with AWS Organizations integration

**Technologies Used:**
- Terraform (IaC)
- AWS Services: IAM, VPC, GuardDuty, Security Hub, CloudTrail, Config, KMS, S3, SNS, Lambda
- Python (Boto3) for monitoring and enforcement

---

## Prerequisites

### Required Tools
```bash
# Terraform (v1.5+)
wget https://releases.hashicorp.com/terraform/1.5.0/terraform_1.5.0_linux_amd64.zip
unzip terraform_1.5.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# AWS CLI (v2)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Python 3.9+ with pip
sudo apt-get install python3.9 python3-pip

# Install Python dependencies
pip3 install boto3 botocore python-dateutil
```

### AWS Account Setup
1. **AWS Account**: Active AWS account with administrator access
2. **IAM User/Role**: Create an IAM user with programmatic access and attach `AdministratorAccess` policy
3. **AWS Credentials Configuration**:
```bash
aws configure
# Enter: Access Key ID, Secret Access Key, Default region (e.g., us-east-1), Output format (json)
```

### Permissions Required
- Full access to: IAM, VPC, EC2, S3, CloudTrail, Config, GuardDuty, Security Hub, KMS, SNS, Lambda, CloudWatch
- For multi-account: AWS Organizations management account access

---

## Architecture Overview

### Security Layers

```
┌─────────────────────────────────────────────────────────────┐
│                     AWS ACCOUNT                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  IDENTITY & ACCESS MANAGEMENT                          │ │
│  │  • IAM Baseline (Password Policy, MFA)                 │ │
│  │  • Service Control Policies (SCPs)                     │ │
│  │  • Cross-Account Roles                                 │ │
│  └────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  NETWORK LAYER                                         │ │
│  │  • Hardened VPC (Multi-AZ)                             │ │
│  │  • Public/Private Subnets                              │ │
│  │  • Security Groups (Least Privilege)                   │ │
│  │  • NACLs, VPC Flow Logs                                │ │
│  └────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  LOGGING & MONITORING                                  │ │
│  │  • CloudTrail (Multi-Region, Encrypted)                │ │
│  │  • AWS Config (Compliance Rules)                       │ │
│  │  • GuardDuty (Threat Detection)                        │ │
│  │  • Security Hub (Centralized Findings)                 │ │
│  └────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  DATA PROTECTION                                       │ │
│  │  • KMS Encryption Keys                                 │ │
│  │  • Encrypted S3 Buckets (Versioning, Logging)          │ │
│  │  • Backup & Recovery                                   │ │
│  └────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  CONTINUOUS COMPLIANCE                                 │ │
│  │  • Python Monitoring Script (Lambda-based)             │ │
│  │  • Automated Remediation                               │ │
│  │  • SNS Alerts for Critical Issues                      │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### CIS Benchmark Controls Implemented

| Control | Description | Service |
|---------|-------------|---------|
| CIS 1.x | IAM policies, password policy, MFA enforcement | IAM |
| CIS 2.x | Logging enabled (CloudTrail multi-region) | CloudTrail |
| CIS 3.x | Monitoring and alerting | CloudWatch, SNS |
| CIS 4.x | Networking security (VPC, Security Groups) | VPC, EC2 |
| CIS 5.x | Data encryption at rest and in transit | KMS, S3 |

---

## Step-by-Step Implementation

### Phase 1: Environment Setup (15 minutes)

**Step 1: Create Project Directory Structure**
```bash
mkdir -p aws-security-baseline
cd aws-security-baseline

mkdir -p {modules,environments,scripts,policies}
mkdir -p modules/{iam,vpc,security-services,kms,s3}
mkdir -p environments/{dev,staging,prod}
```

**Step 2: Initialize Git Repository**
```bash
git init
echo "*.tfstate" >> .gitignore
echo "*.tfstate.backup" >> .gitignore
echo ".terraform/" >> .gitignore
echo "*.tfvars" >> .gitignore
echo "__pycache__/" >> .gitignore
```

### Phase 2: Terraform Module Development (1-2 hours)

**Step 3: Create Terraform Backend Configuration**

Create `environments/prod/backend.tf`:
```hcl
terraform {
  backend "s3" {
    bucket         = "your-terraform-state-bucket"
    key            = "security-baseline/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}
```

**Step 4: Build Core Modules**

The following sections contain the Terraform code for each module. Create these files in your project structure.

### Phase 3: Python Monitoring Setup (1 hour)

**Step 5: Deploy Monitoring Lambda Function**

Create the Python monitoring script (covered in detail in the Python section below).

**Step 6: Configure SNS Notifications**

Set up SNS topics for security alerts.

### Phase 4: Deployment (30 minutes)

**Step 7: Deploy Infrastructure**
```bash
cd environments/prod
terraform init
terraform plan -out=tfplan
terraform apply tfplan
```

**Step 8: Verify Deployment**
```bash
# Check GuardDuty is enabled
aws guardduty list-detectors --region us-east-1

# Check Security Hub is enabled
aws securityhub describe-hub --region us-east-1

# Check CloudTrail is logging
aws cloudtrail describe-trails --region us-east-1
```

### Phase 5: Testing & Validation (30 minutes)

**Step 9: Run Compliance Checks**

Execute the Python monitoring script to verify CIS compliance.

**Step 10: Review Security Hub Findings**

Check AWS Security Hub console for any critical findings.

---

## Terraform Module Structure

### Complete Terraform Code

Below is the comprehensive Terraform code organized by module. Each module is self-contained and reusable.

---

#### Module 1: IAM Baseline (`modules/iam/main.tf`)

```hcl
# IAM Password Policy (CIS 1.5-1.11)
resource "aws_iam_account_password_policy" "strict" {
  minimum_password_length        = 14
  require_lowercase_characters   = true
  require_uppercase_characters   = true
  require_numbers                = true
  require_symbols                = true
  allow_users_to_change_password = true
  max_password_age               = 90
  password_reuse_prevention      = 24
  hard_expiry                    = false
}

# IAM Access Analyzer (CIS 1.20)
resource "aws_accessanalyzer_analyzer" "account" {
  analyzer_name = "account-analyzer"
  type          = "ACCOUNT"

  tags = {
    Name        = "Account Access Analyzer"
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

# IAM Role for Security Services
resource "aws_iam_role" "security_services" {
  name = "${var.environment}-security-services-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = [
            "config.amazonaws.com",
            "cloudtrail.amazonaws.com",
            "securityhub.amazonaws.com"
          ]
        }
      }
    ]
  })

  tags = var.tags
}

# IAM Policy for Security Services
resource "aws_iam_role_policy_attachment" "security_services" {
  role       = aws_iam_role.security_services.name
  policy_arn = "arn:aws:iam::aws:policy/SecurityAudit"
}

# Support Role (CIS 1.20)
resource "aws_iam_role" "support" {
  name = "${var.environment}-support-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          AWS = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:root"
        }
        Condition = {
          Bool = {
            "aws:MultiFactorAuthPresent" = "true"
          }
        }
      }
    ]
  })

  tags = var.tags
}

resource "aws_iam_role_policy_attachment" "support" {
  role       = aws_iam_role.support.name
  policy_arn = "arn:aws:iam::aws:policy/AWSSupportAccess"
}

data "aws_caller_identity" "current" {}
```

**Variables (`modules/iam/variables.tf`):**
```hcl
variable "environment" {
  description = "Environment name"
  type        = string
}

variable "tags" {
  description = "Common tags"
  type        = map(string)
  default     = {}
}
```

**Outputs (`modules/iam/outputs.tf`):**
```hcl
output "security_services_role_arn" {
  description = "ARN of security services IAM role"
  value       = aws_iam_role.security_services.arn
}

output "analyzer_arn" {
  description = "ARN of IAM Access Analyzer"
  value       = aws_accessanalyzer_analyzer.account.arn
}
```

---

#### Module 2: VPC with Security Hardening (`modules/vpc/main.tf`)

```hcl
# VPC (CIS 4.1)
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-secure-vpc"
    }
  )
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-igw"
    }
  )
}

# Public Subnets (Multi-AZ)
resource "aws_subnet" "public" {
  count                   = length(var.availability_zones)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4, count.index)
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = false  # CIS 4.3: Disable auto-assign public IP

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-public-subnet-${count.index + 1}"
      Type = "Public"
    }
  )
}

# Private Subnets (Multi-AZ)
resource "aws_subnet" "private" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 4, count.index + length(var.availability_zones))
  availability_zone = var.availability_zones[count.index]

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-private-subnet-${count.index + 1}"
      Type = "Private"
    }
  )
}

# Elastic IPs for NAT Gateways
resource "aws_eip" "nat" {
  count  = length(var.availability_zones)
  domain = "vpc"

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-nat-eip-${count.index + 1}"
    }
  )

  depends_on = [aws_internet_gateway.main]
}

# NAT Gateways (Multi-AZ for HA)
resource "aws_nat_gateway" "main" {
  count         = length(var.availability_zones)
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-nat-gw-${count.index + 1}"
    }
  )

  depends_on = [aws_internet_gateway.main]
}

# Public Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-public-rt"
    }
  )
}

# Public Subnet Route Table Associations
resource "aws_route_table_association" "public" {
  count          = length(aws_subnet.public)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

# Private Route Tables (one per AZ)
resource "aws_route_table" "private" {
  count  = length(var.availability_zones)
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-private-rt-${count.index + 1}"
    }
  )
}

# Private Subnet Route Table Associations
resource "aws_route_table_association" "private" {
  count          = length(aws_subnet.private)
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}

# VPC Flow Logs (CIS 3.9)
resource "aws_flow_log" "main" {
  iam_role_arn    = var.flow_log_role_arn
  log_destination = var.flow_log_destination_arn
  traffic_type    = "ALL"
  vpc_id          = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-vpc-flow-logs"
    }
  )
}

# Default Security Group - Deny All (CIS 4.4)
resource "aws_default_security_group" "default" {
  vpc_id = aws_vpc.main.id

  # No ingress or egress rules = deny all

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-default-sg-deny-all"
    }
  )
}

# Security Group: Web Tier (Example Template)
resource "aws_security_group" "web" {
  name_prefix = "${var.environment}-web-"
  description = "Security group for web tier"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "HTTPS from internet"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTP from internet (redirect to HTTPS)"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "Allow all outbound"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-web-sg"
    }
  )

  lifecycle {
    create_before_destroy = true
  }
}

# Security Group: Application Tier
resource "aws_security_group" "app" {
  name_prefix = "${var.environment}-app-"
  description = "Security group for application tier"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "From web tier"
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.web.id]
  }

  egress {
    description = "Allow all outbound"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-app-sg"
    }
  )

  lifecycle {
    create_before_destroy = true
  }
}

# Security Group: Database Tier
resource "aws_security_group" "db" {
  name_prefix = "${var.environment}-db-"
  description = "Security group for database tier"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "From app tier"
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }

  egress {
    description = "Allow all outbound"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-db-sg"
    }
  )

  lifecycle {
    create_before_destroy = true
  }
}

# Network ACL for additional defense in depth
resource "aws_network_acl" "main" {
  vpc_id     = aws_vpc.main.id
  subnet_ids = concat(aws_subnet.public[*].id, aws_subnet.private[*].id)

  # Allow all traffic by default (adjust based on requirements)
  ingress {
    protocol   = -1
    rule_no    = 100
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 0
    to_port    = 0
  }

  egress {
    protocol   = -1
    rule_no    = 100
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 0
    to_port    = 0
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-main-nacl"
    }
  )
}
```

**Variables (`modules/vpc/variables.tf`):**
```hcl
variable "environment" {
  description = "Environment name"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

variable "flow_log_role_arn" {
  description = "IAM role ARN for VPC Flow Logs"
  type        = string
}

variable "flow_log_destination_arn" {
  description = "CloudWatch log group ARN for VPC Flow Logs"
  type        = string
}

variable "tags" {
  description = "Common tags"
  type        = map(string)
  default     = {}
}
```

**Outputs (`modules/vpc/outputs.tf`):**
```hcl
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "Private subnet IDs"
  value       = aws_subnet.private[*].id
}

output "security_group_web_id" {
  description = "Web tier security group ID"
  value       = aws_security_group.web.id
}

output "security_group_app_id" {
  description = "App tier security group ID"
  value       = aws_security_group.app.id
}

output "security_group_db_id" {
  description = "Database tier security group ID"
  value       = aws_security_group.db.id
}
```

---

#### Module 3: Security Services (`modules/security-services/main.tf`)

```hcl
# CloudTrail (CIS 2.1-2.7)
resource "aws_cloudtrail" "main" {
  name                          = "${var.environment}-cloudtrail"
  s3_bucket_name                = var.cloudtrail_bucket_name
  include_global_service_events = true
  is_multi_region_trail         = true
  enable_log_file_validation    = true
  kms_key_id                    = var.kms_key_arn

  event_selector {
    read_write_type           = "All"
    include_management_events = true

    data_resource {
      type   = "AWS::S3::Object"
      values = ["arn:aws:s3:::*/"]
    }

    data_resource {
      type   = "AWS::Lambda::Function"
      values = ["arn:aws:lambda:*:${data.aws_caller_identity.current.account_id}:function/*"]
    }
  }

  insight_selector {
    insight_type = "ApiCallRateInsight"
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-cloudtrail"
    }
  )

  depends_on = [var.cloudtrail_bucket_policy_dependency]
}

# CloudWatch Log Group for CloudTrail
resource "aws_cloudwatch_log_group" "cloudtrail" {
  name              = "/aws/cloudtrail/${var.environment}"
  retention_in_days = 90
  kms_key_id        = var.kms_key_arn

  tags = var.tags
}

# IAM Role for CloudTrail CloudWatch Logs
resource "aws_iam_role" "cloudtrail_cloudwatch" {
  name = "${var.environment}-cloudtrail-cloudwatch-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "cloudtrail.amazonaws.com"
        }
      }
    ]
  })

  tags = var.tags
}

resource "aws_iam_role_policy" "cloudtrail_cloudwatch" {
  name = "${var.environment}-cloudtrail-cloudwatch-policy"
  role = aws_iam_role.cloudtrail_cloudwatch.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "${aws_cloudwatch_log_group.cloudtrail.arn}:*"
      }
    ]
  })
}

# AWS Config (CIS 2.5)
resource "aws_config_configuration_recorder" "main" {
  name     = "${var.environment}-config-recorder"
  role_arn = aws_iam_role.config.arn

  recording_group {
    all_supported                 = true
    include_global_resource_types = true
  }
}

resource "aws_config_delivery_channel" "main" {
  name           = "${var.environment}-config-delivery"
  s3_bucket_name = var.config_bucket_name
  s3_key_prefix  = "config"

  snapshot_delivery_properties {
    delivery_frequency = "TwentyFour_Hours"
  }

  depends_on = [aws_config_configuration_recorder.main]
}

resource "aws_config_configuration_recorder_status" "main" {
  name       = aws_config_configuration_recorder.main.name
  is_enabled = true

  depends_on = [aws_config_delivery_channel.main]
}

# IAM Role for AWS Config
resource "aws_iam_role" "config" {
  name = "${var.environment}-config-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "config.amazonaws.com"
        }
      }
    ]
  })

  tags = var.tags
}

resource "aws_iam_role_policy_attachment" "config" {
  role       = aws_iam_role.config.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/ConfigRole"
}

resource "aws_iam_role_policy" "config_s3" {
  name = "${var.environment}-config-s3-policy"
  role = aws_iam_role.config.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:PutObject",
          "s3:GetBucketLocation"
        ]
        Resource = [
          "arn:aws:s3:::${var.config_bucket_name}/*",
          "arn:aws:s3:::${var.config_bucket_name}"
        ]
      }
    ]
  })
}

# AWS Config Rules (CIS Compliance)
resource "aws_config_config_rule" "encrypted_volumes" {
  name = "encrypted-volumes"

  source {
    owner             = "AWS"
    source_identifier = "ENCRYPTED_VOLUMES"
  }

  depends_on = [aws_config_configuration_recorder.main]
}

resource "aws_config_config_rule" "root_account_mfa" {
  name = "root-account-mfa-enabled"

  source {
    owner             = "AWS"
    source_identifier = "ROOT_ACCOUNT_MFA_ENABLED"
  }

  depends_on = [aws_config_configuration_recorder.main]
}

resource "aws_config_config_rule" "iam_password_policy" {
  name = "iam-password-policy"

  source {
    owner             = "AWS"
    source_identifier = "IAM_PASSWORD_POLICY"
  }

  input_parameters = jsonencode({
    RequireUppercaseCharacters = true
    RequireLowercaseCharacters = true
    RequireSymbols             = true
    RequireNumbers             = true
    MinimumPasswordLength      = 14
    PasswordReusePrevention    = 24
    MaxPasswordAge             = 90
  })

  depends_on = [aws_config_configuration_recorder.main]
}

resource "aws_config_config_rule" "s3_bucket_public_read_prohibited" {
  name = "s3-bucket-public-read-prohibited"

  source {
    owner             = "AWS"
    source_identifier = "S3_BUCKET_PUBLIC_READ_PROHIBITED"
  }

  depends_on = [aws_config_configuration_recorder.main]
}

resource "aws_config_config_rule" "s3_bucket_public_write_prohibited" {
  name = "s3-bucket-public-write-prohibited"

  source {
    owner             = "AWS"
    source_identifier = "S3_BUCKET_PUBLIC_WRITE_PROHIBITED"
  }

  depends_on = [aws_config_configuration_recorder.main]
}

# GuardDuty (Threat Detection)
resource "aws_guardduty_detector" "main" {
  enable = true

  datasources {
    s3_logs {
      enable = true
    }
    kubernetes {
      audit_logs {
        enable = true
      }
    }
    malware_protection {
      scan_ec2_instance_with_findings {
        ebs_volumes {
          enable = true
        }
      }
    }
  }

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-guardduty"
    }
  )
}

# Security Hub
resource "aws_securityhub_account" "main" {}

resource "aws_securityhub_standards_subscription" "cis" {
  depends_on    = [aws_securityhub_account.main]
  standards_arn = "arn:aws:securityhub:::ruleset/cis-aws-foundations-benchmark/v/1.2.0"
}

resource "aws_securityhub_standards_subscription" "pci_dss" {
  depends_on    = [aws_securityhub_account.main]
  standards_arn = "arn:aws:securityhub:${data.aws_region.current.name}::standards/pci-dss/v/3.2.1"
}

# SNS Topic for Security Alerts (CIS 3.x)
resource "aws_sns_topic" "security_alerts" {
  name              = "${var.environment}-security-alerts"
  kms_master_key_id = var.kms_key_id

  tags = var.tags
}

resource "aws_sns_topic_subscription" "security_alerts_email" {
  topic_arn = aws_sns_topic.security_alerts.arn
  protocol  = "email"
  endpoint  = var.alert_email
}

# CloudWatch Alarms for CIS Monitoring (CIS 3.1-3.14)
resource "aws_cloudwatch_log_metric_filter" "unauthorized_api_calls" {
  name           = "unauthorized-api-calls"
  log_group_name = aws_cloudwatch_log_group.cloudtrail.name
  pattern        = "{ ($.errorCode = \"*UnauthorizedOperation\") || ($.errorCode = \"AccessDenied*\") }"

  metric_transformation {
    name      = "UnauthorizedAPICalls"
    namespace = "${var.environment}/CISBenchmark"
    value     = "1"
  }
}

resource "aws_cloudwatch_metric_alarm" "unauthorized_api_calls" {
  alarm_name          = "${var.environment}-unauthorized-api-calls"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = 1
  metric_name         = "UnauthorizedAPICalls"
  namespace           = "${var.environment}/CISBenchmark"
  period              = 300
  statistic           = "Sum"
  threshold           = 1
  alarm_description   = "Triggered by unauthorized API calls"
  alarm_actions       = [aws_sns_topic.security_alerts.arn]
  treat_missing_data  = "notBreaching"
}

resource "aws_cloudwatch_log_metric_filter" "root_account_usage" {
  name           = "root-account-usage"
  log_group_name = aws_cloudwatch_log_group.cloudtrail.name
  pattern        = "{ $.userIdentity.type = \"Root\" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != \"AwsServiceEvent\" }"

  metric_transformation {
    name      = "RootAccountUsage"
    namespace = "${var.environment}/CISBenchmark"
    value     = "1"
  }
}

resource "aws_cloudwatch_metric_alarm" "root_account_usage" {
  alarm_name          = "${var.environment}-root-account-usage"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = 1
  metric_name         = "RootAccountUsage"
  namespace           = "${var.environment}/CISBenchmark"
  period              = 60
  statistic           = "Sum"
  threshold           = 1
  alarm_description   = "Triggered by root account usage"
  alarm_actions       = [aws_sns_topic.security_alerts.arn]
  treat_missing_data  = "notBreaching"
}

resource "aws_cloudwatch_log_metric_filter" "iam_policy_changes" {
  name           = "iam-policy-changes"
  log_group_name = aws_cloudwatch_log_group.cloudtrail.name
  pattern        = "{($.eventName=DeleteGroupPolicy)||($.eventName=DeleteRolePolicy)||($.eventName=DeleteUserPolicy)||($.eventName=PutGroupPolicy)||($.eventName=PutRolePolicy)||($.eventName=PutUserPolicy)||($.eventName=CreatePolicy)||($.eventName=DeletePolicy)||($.eventName=CreatePolicyVersion)||($.eventName=DeletePolicyVersion)||($.eventName=AttachRolePolicy)||($.eventName=DetachRolePolicy)||($.eventName=AttachUserPolicy)||($.eventName=DetachUserPolicy)||($.eventName=AttachGroupPolicy)||($.eventName=DetachGroupPolicy)}"

  metric_transformation {
    name      = "IAMPolicyChanges"
    namespace = "${var.environment}/CISBenchmark"
    value     = "1"
  }
}

resource "aws_cloudwatch_metric_alarm" "iam_policy_changes" {
  alarm_name          = "${var.environment}-iam-policy-changes"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = 1
  metric_name         = "IAMPolicyChanges"
  namespace           = "${var.environment}/CISBenchmark"
  period              = 300
  statistic           = "Sum"
  threshold           = 1
  alarm_description   = "Triggered by IAM policy changes"
  alarm_actions       = [aws_sns_topic.security_alerts.arn]
  treat_missing_data  = "notBreaching"
}

data "aws_caller_identity" "current" {}
data "aws_region" "current" {}
```

**Variables (`modules/security-services/variables.tf`):**
```hcl
variable "environment" {
  description = "Environment name"
  type        = string
}

variable "cloudtrail_bucket_name" {
  description = "S3 bucket name for CloudTrail logs"
  type        = string
}

variable "config_bucket_name" {
  description = "S3 bucket name for AWS Config"
  type        = string
}

variable "kms_key_arn" {
  description = "KMS key ARN for encryption"
  type        = string
}

variable "kms_key_id" {
  description = "KMS key ID for encryption"
  type        = string
}

variable "alert_email" {
  description = "Email address for security alerts"
  type        = string
}

variable "cloudtrail_bucket_policy_dependency" {
  description = "Dependency to ensure bucket policy is created first"
  type        = any
  default     = null
}

variable "tags" {
  description = "Common tags"
  type        = map(string)
  default     = {}
}
```

**Outputs (`modules/security-services/outputs.tf`):**
```hcl
output "cloudtrail_arn" {
  description = "CloudTrail ARN"
  value       = aws_cloudtrail.main.arn
}

output "config_recorder_id" {
  description = "Config recorder ID"
  value       = aws_config_configuration_recorder.main.id
}

output "guardduty_detector_id" {
  description = "GuardDuty detector ID"
  value       = aws_guardduty_detector.main.id
}

output "sns_topic_arn" {
  description = "SNS topic ARN for security alerts"
  value       = aws_sns_topic.security_alerts.arn
}
```

---

#### Module 4: KMS Keys (`modules/kms/main.tf`)

```hcl
# KMS Key for Data Encryption (CIS 2.7, 3.7)
resource "aws_kms_key" "main" {
  description             = "${var.environment} KMS key for data encryption"
  deletion_window_in_days = 30
  enable_key_rotation     = true

  tags = merge(
    var.tags,
    {
      Name = "${var.environment}-kms-key"
    }
  )
}

resource "aws_kms_alias" "main" {
  name          = "alias/${var.environment}-main-key"
  target_key_id = aws_kms_key.main.key_id
}

# KMS Key Policy
resource "aws_kms_key_policy" "main" {
  key_id = aws_kms_key.main.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "Enable IAM User Permissions"
        Effect = "Allow"
        Principal = {
          AWS = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:root"
        }
        Action   = "kms:*"
        Resource = "*"
      },
      {
        Sid    = "Allow CloudTrail to encrypt logs"
        Effect = "Allow"
        Principal = {
          Service = "cloudtrail.amazonaws.com"
        }
        Action = [
          "kms:GenerateDataKey*",
          "kms:DecryptDataKey"
        ]
        Resource = "*"
        Condition = {
          StringLike = {
            "kms:EncryptionContext:aws:cloudtrail:arn" = "arn:aws:cloudtrail:*:${data.aws_caller_identity.current.account_id}:trail/*"
          }
        }
      },
      {
        Sid    = "Allow Config to use the key"
        Effect = "Allow"
        Principal = {
          Service = "config.amazonaws.com"
        }
        Action = [
          "kms:Decrypt",
          "kms:GenerateDataKey"
        ]
        Resource = "*"
      },
      {
        Sid    = "Allow CloudWatch Logs"
        Effect = "Allow"
        Principal = {
          Service = "logs.${data.aws_region.current.name}.amazonaws.com"
        }
        Action = [
          "kms:Encrypt",
          "kms:Decrypt",
          "kms:ReEncrypt*",
          "kms:GenerateDataKey*",
          "kms:CreateGrant",
          "kms:DescribeKey"
        ]
        Resource = "*"
        Condition = {
          ArnLike = {
            "kms:EncryptionContext:aws:logs:arn" = "arn:aws:logs:${data.aws_region.current.name}:${data.aws_caller_identity.current.account_id}:log-group:*"
          }
        }
      },
      {
        Sid    = "Allow SNS to use the key"
        Effect = "Allow"
        Principal = {
          Service = "sns.amazonaws.com"
        }
        Action = [
          "kms:Decrypt",
          "kms:GenerateDataKey"
        ]
        Resource = "*"
      }
    ]
  })
}

data "aws_caller_identity" "current" {}
data "aws_region" "current" {}
```

**Variables (`modules/kms/variables.tf`):**
```hcl
variable "environment" {
  description = "Environment name"
  type        = string
}

variable "tags" {
  description = "Common tags"
  type        = map(string)
  default     = {}
}
```

**Outputs (`modules/kms/outputs.tf`):**
```hcl
output "key_id" {
  description = "KMS key ID"
  value       = aws_kms_key.main.key_id
}

output "key_arn" {
  description = "KMS key ARN"
  value       = aws_kms_key.main.arn
}

output "key_alias" {
  description = "KMS key alias"
  value       = aws_kms_alias.main.name
}
```

---

#### Module 5: S3 Secure Buckets (`modules/s3/main.tf`)

```hcl
# S3 Bucket for CloudTrail (CIS 2.3, 2.6)
resource "aws_s3_bucket" "cloudtrail" {
  bucket = "${var.environment}-cloudtrail-logs-${data.aws_caller_identity.current.account_id}"

  tags = merge(
    var.tags,
    {
      Name    = "${var.environment}-cloudtrail-logs"
      Purpose = "CloudTrail Logs"
    }
  )
}

resource "aws_s3_bucket_versioning" "cloudtrail" {
  bucket = aws_s3_bucket.cloudtrail.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "cloudtrail" {
  bucket = aws_s3_bucket.cloudtrail.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = var.kms_key_arn
    }
  }
}

resource "aws_s3_bucket_public_access_block" "cloudtrail" {
  bucket = aws_s3_bucket.cloudtrail.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_logging" "cloudtrail" {
  bucket = aws_s3_bucket.cloudtrail.id

  target_bucket = aws_s3_bucket.access_logs.id
  target_prefix = "cloudtrail-bucket/"
}

resource "aws_s3_bucket_lifecycle_configuration" "cloudtrail" {
  bucket = aws_s3_bucket.cloudtrail.id

  rule {
    id     = "transition-to-glacier"
    status = "Enabled"

    transition {
      days          = 90
      storage_class = "GLACIER"
    }

    expiration {
      days = 365
    }
  }
}

resource "aws_s3_bucket_policy" "cloudtrail" {
  bucket = aws_s3_bucket.cloudtrail.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "AWSCloudTrailAclCheck"
        Effect = "Allow"
        Principal = {
          Service = "cloudtrail.amazonaws.com"
        }
        Action   = "s3:GetBucketAcl"
        Resource = aws_s3_bucket.cloudtrail.arn
      },
      {
        Sid    = "AWSCloudTrailWrite"
        Effect = "Allow"
        Principal = {
          Service = "cloudtrail.amazonaws.com"
        }
        Action   = "s3:PutObject"
        Resource = "${aws_s3_bucket.cloudtrail.arn}/*"
        Condition = {
          StringEquals = {
            "s3:x-amz-acl" = "bucket-owner-full-control"
          }
        }
      },
      {
        Sid    = "DenyUnencryptedObjectUploads"
        Effect = "Deny"
        Principal = "*"
        Action = "s3:PutObject"
        Resource = "${aws_s3_bucket.cloudtrail.arn}/*"
        Condition = {
          StringNotEquals = {
            "s3:x-amz-server-side-encryption" = "aws:kms"
          }
        }
      },
      {
        Sid    = "DenyInsecureTransport"
        Effect = "Deny"
        Principal = "*"
        Action = "s3:*"
        Resource = [
          aws_s3_bucket.cloudtrail.arn,
          "${aws_s3_bucket.cloudtrail.arn}/*"
        ]
        Condition = {
          Bool = {
            "aws:SecureTransport" = "false"
          }
        }
      }
    ]
  })
}

# S3 Bucket for AWS Config
resource "aws_s3_bucket" "config" {
  bucket = "${var.environment}-config-logs-${data.aws_caller_identity.current.account_id}"

  tags = merge(
    var.tags,
    {
      Name    = "${var.environment}-config-logs"
      Purpose = "AWS Config"
    }
  )
}

resource "aws_s3_bucket_versioning" "config" {
  bucket = aws_s3_bucket.config.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "config" {
  bucket = aws_s3_bucket.config.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = var.kms_key_arn
    }
  }
}

resource "aws_s3_bucket_public_access_block" "config" {
  bucket = aws_s3_bucket.config.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_policy" "config" {
  bucket = aws_s3_bucket.config.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "AWSConfigBucketPermissionsCheck"
        Effect = "Allow"
        Principal = {
          Service = "config.amazonaws.com"
        }
        Action   = "s3:GetBucketAcl"
        Resource = aws_s3_bucket.config.arn
      },
      {
        Sid    = "AWSConfigBucketExistenceCheck"
        Effect = "Allow"
        Principal = {
          Service = "config.amazonaws.com"
        }
        Action   = "s3:ListBucket"
        Resource = aws_s3_bucket.config.arn
      },
      {
        Sid    = "AWSConfigBucketPutObject"
        Effect = "Allow"
        Principal = {
          Service = "config.amazonaws.com"
        }
        Action   = "s3:PutObject"
        Resource = "${aws_s3_bucket.config.arn}/*"
        Condition = {
          StringEquals = {
            "s3:x-amz-acl" = "bucket-owner-full-control"
          }
        }
      },
      {
        Sid    = "DenyInsecureTransport"
        Effect = "Deny"
        Principal = "*"
        Action = "s3:*"
        Resource = [
          aws_s3_bucket.config.arn,
          "${aws_s3_bucket.config.arn}/*"
        ]
        Condition = {
          Bool = {
            "aws:SecureTransport" = "false"
          }
        }
      }
    ]
  })
}

# S3 Bucket for Access Logs (CIS 2.6)
resource "aws_s3_bucket" "access_logs" {
  bucket = "${var.environment}-access-logs-${data.aws_caller_identity.current.account_id}"

  tags = merge(
    var.tags,
    {
      Name    = "${var.environment}-access-logs"
      Purpose = "S3 Access Logs"
    }
  )
}

resource "aws_s3_bucket_versioning" "access_logs" {
  bucket = aws_s3_bucket.access_logs.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "access_logs" {
  bucket = aws_s3_bucket.access_logs.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "access_logs" {
  bucket = aws_s3_bucket.access_logs.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_lifecycle_configuration" "access_logs" {
  bucket = aws_s3_bucket.access_logs.id

  rule {
    id     = "expire-old-logs"
    status = "Enabled"

    expiration {
      days = 90
    }
  }
}

# CloudWatch Log Group for VPC Flow Logs
resource "aws_cloudwatch_log_group" "flow_logs" {
  name              = "/aws/vpc/flowlogs/${var.environment}"
  retention_in_days = 90
  kms_key_id        = var.kms_key_arn

  tags = var.tags
}

# IAM Role for VPC Flow Logs
resource "aws_iam_role" "flow_logs" {
  name = "${var.environment}-vpc-flow-logs-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "vpc-flow-logs.amazonaws.com"
        }
      }
    ]
  })

  tags = var.tags
}

resource "aws_iam_role_policy" "flow_logs" {
  name = "${var.environment}-vpc-flow-logs-policy"
  role = aws_iam_role.flow_logs.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents",
          "logs:DescribeLogGroups",
          "logs:DescribeLogStreams"
        ]
        Resource = "*"
      }
    ]
  })
}

data "aws_caller_identity" "current" {}
```

**Variables (`modules/s3/variables.tf`):**
```hcl
variable "environment" {
  description = "Environment name"
  type        = string
}

variable "kms_key_arn" {
  description = "KMS key ARN for encryption"
  type        = string
}

variable "tags" {
  description = "Common tags"
  type        = map(string)
  default     = {}
}
```

**Outputs (`modules/s3/outputs.tf`):**
```hcl
output "cloudtrail_bucket_name" {
  description = "CloudTrail S3 bucket name"
  value       = aws_s3_bucket.cloudtrail.id
}

output "cloudtrail_bucket_arn" {
  description = "CloudTrail S3 bucket ARN"
  value       = aws_s3_bucket.cloudtrail.arn
}

output "config_bucket_name" {
  description = "Config S3 bucket name"
  value       = aws_s3_bucket.config.id
}

output "config_bucket_arn" {
  description = "Config S3 bucket ARN"
  value       = aws_s3_bucket.config.arn
}

output "flow_log_role_arn" {
  description = "IAM role ARN for VPC Flow Logs"
  value       = aws_iam_role.flow_logs.arn
}

output "flow_log_destination_arn" {
  description = "CloudWatch log group ARN for VPC Flow Logs"
  value       = aws_cloudwatch_log_group.flow_logs.arn
}

output "cloudtrail_bucket_policy_id" {
  description = "CloudTrail bucket policy ID for dependency management"
  value       = aws_s3_bucket_policy.cloudtrail.id
}
```

---

#### Root Module Configuration (`environments/prod/main.tf`)

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Environment = var.environment
      Project     = "AWS-Security-Baseline"
      ManagedBy   = "Terraform"
      Owner       = var.owner
    }
  }
}

locals {
  common_tags = {
    Environment = var.environment
    Project     = "AWS-Security-Baseline"
    ManagedBy   = "Terraform"
    Owner       = var.owner
  }
}

# KMS Module
module "kms" {
  source = "../../modules/kms"

  environment = var.environment
  tags        = local.common_tags
}

# S3 Module
module "s3" {
  source = "../../modules/s3"

  environment = var.environment
  kms_key_arn = module.kms.key_arn
  tags        = local.common_tags
}

# IAM Module
module "iam" {
  source = "../../modules/iam"

  environment = var.environment
  tags        = local.common_tags
}

# VPC Module
module "vpc" {
  source = "../../modules/vpc"

  environment                 = var.environment
  vpc_cidr                    = var.vpc_cidr
  availability_zones          = var.availability_zones
  flow_log_role_arn           = module.s3.flow_log_role_arn
  flow_log_destination_arn    = module.s3.flow_log_destination_arn
  tags                        = local.common_tags
}

# Security Services Module
module "security_services" {
  source = "../../modules/security-services"

  environment                          = var.environment
  cloudtrail_bucket_name               = module.s3.cloudtrail_bucket_name
  config_bucket_name                   = module.s3.config_bucket_name
  kms_key_arn                          = module.kms.key_arn
  kms_key_id                           = module.kms.key_id
  alert_email                          = var.alert_email
  cloudtrail_bucket_policy_dependency  = module.s3.cloudtrail_bucket_policy_id
  tags                                 = local.common_tags
}
```

**Variables (`environments/prod/variables.tf`):**
```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "prod"
}

variable "owner" {
  description = "Owner email or team"
  type        = string
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

variable "alert_email" {
  description = "Email for security alerts"
  type        = string
}
```

**Outputs (`environments/prod/outputs.tf`):**
```hcl
output "vpc_id" {
  description = "VPC ID"
  value       = module.vpc.vpc_id
}

output "kms_key_id" {
  description = "KMS key ID"
  value       = module.kms.key_id
}

output "cloudtrail_arn" {
  description = "CloudTrail ARN"
  value       = module.security_services.cloudtrail_arn
}

output "guardduty_detector_id" {
  description = "GuardDuty detector ID"
  value       = module.security_services.guardduty_detector_id
}

output "security_alerts_topic_arn" {
  description = "SNS topic ARN for security alerts"
  value       = module.security_services.sns_topic_arn
}
```

**Terraform Variables File (`environments/prod/terraform.tfvars`):**
```hcl
aws_region         = "us-east-1"
environment        = "prod"
owner              = "security-team@example.com"
vpc_cidr           = "10.0.0.0/16"
availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]
alert_email        = "security-alerts@example.com"
```

---

## Python Monitoring & Enforcement

### Comprehensive CIS Compliance Monitoring Script

This Python script monitors your AWS environment and enforces CIS benchmark standards. It can run as a Lambda function on a schedule or as a standalone script.

**File: `scripts/cis_compliance_monitor.py`**

```python
#!/usr/bin/env python3
"""
AWS CIS Benchmark Compliance Monitor
Monitors AWS environment and enforces CIS AWS Foundations Benchmark standards
"""

import boto3
import json
import logging
from datetime import datetime, timedelta
from typing import Dict, List, Any
from botocore.exceptions import ClientError

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

class CISComplianceMonitor:
    """Monitor and enforce CIS AWS Foundations Benchmark compliance"""
    
    def __init__(self, region: str = 'us-east-1'):
        self.region = region
        self.iam_client = boto3.client('iam')
        self.ec2_client = boto3.client('ec2', region_name=region)
        self.s3_client = boto3.client('s3')
        self.cloudtrail_client = boto3.client('cloudtrail', region_name=region)
        self.config_client = boto3.client('config', region_name=region)
        self.guardduty_client = boto3.client('guardduty', region_name=region)
        self.kms_client = boto3.client('kms', region_name=region)
        self.sns_client = boto3.client('sns', region_name=region)
        self.securityhub_client = boto3.client('securityhub', region_name=region)
        self.sts_client = boto3.client('sts')
        
        self.account_id = self.sts_client.get_caller_identity()['Account']
        self.findings = []
        
    def run_all_checks(self) -> List[Dict[str, Any]]:
        """Run all CIS compliance checks"""
        logger.info("Starting CIS compliance checks...")
        
        # IAM Checks (CIS 1.x)
        self.check_root_account_mfa()
        self.check_iam_password_policy()
        self.check_root_access_keys()
        self.check_iam_user_mfa()
        self.check_unused_credentials()
        self.check_access_keys_rotation()
        
        # Logging Checks (CIS 2.x)
        self.check_cloudtrail_enabled()
        self.check_cloudtrail_log_validation()
        self.check_cloudtrail_encryption()
        self.check_cloudtrail_integration()
        self.check_s3_bucket_logging()
        self.check_config_enabled()
        
        # Monitoring Checks (CIS 3.x)
        self.check_cloudwatch_alarms()
        
        # Networking Checks (CIS 4.x)
        self.check_security_groups()
        self.check_default_vpc()
        self.check_vpc_flow_logs()
        
        # Data Protection (CIS 5.x)
        self.check_s3_bucket_encryption()
        self.check_ebs_encryption()
        self.check_rds_encryption()
        
        # Additional Security Services
        self.check_guardduty_enabled()
        self.check_securityhub_enabled()
        
        logger.info(f"Compliance check completed. Found {len(self.findings)} issues.")
        return self.findings
    
    def add_finding(self, severity: str, control: str, description: str, 
                    resource: str = None, remediation: str = None):
        """Add a compliance finding"""
        finding = {
            'timestamp': datetime.utcnow().isoformat(),
            'severity': severity,
            'control': control,
            'description': description,
            'resource': resource,
            'remediation': remediation
        }
        self.findings.append(finding)
        logger.warning(f"[{severity}] {control}: {description}")
    
    # IAM Checks (CIS 1.x)
    
    def check_root_account_mfa(self):
        """CIS 1.5: Ensure MFA is enabled for root account"""
        try:
            summary = self.iam_client.get_account_summary()
            if summary['SummaryMap']['AccountMFAEnabled'] == 0:
                self.add_finding(
                    'CRITICAL',
                    'CIS 1.5',
                    'Root account does not have MFA enabled',
                    'Root Account',
                    'Enable MFA for root account via AWS Console'
                )
        except ClientError as e:
            logger.error(f"Error checking root MFA: {e}")
    
    def check_iam_password_policy(self):
        """CIS 1.5-1.11: Check IAM password policy"""
        try:
            policy = self.iam_client.get_account_password_policy()['PasswordPolicy']
            
            checks = {
                'MinimumPasswordLength': (14, '>='),
                'RequireUppercaseCharacters': (True, '=='),
                'RequireLowercaseCharacters': (True, '=='),
                'RequireNumbers': (True, '=='),
                'RequireSymbols': (True, '=='),
                'MaxPasswordAge': (90, '<='),
                'PasswordReusePrevention': (24, '>='),
            }
            
            for key, (expected, operator) in checks.items():
                actual = policy.get(key)
                
                if operator == '>=' and actual < expected:
                    self.add_finding(
                        'HIGH',
                        f'CIS 1.{key}',
                        f'Password policy {key} is {actual}, should be >= {expected}',
                        'IAM Password Policy',
                        f'Update IAM password policy to set {key} to {expected} or higher'
                    )
                elif operator == '<=' and actual > expected:
                    self.add_finding(
                        'HIGH',
                        f'CIS 1.{key}',
                        f'Password policy {key} is {actual}, should be <= {expected}',
                        'IAM Password Policy',
                        f'Update IAM password policy to set {key} to {expected} or lower'
                    )
                elif operator == '==' and actual != expected:
                    self.add_finding(
                        'HIGH',
                        f'CIS 1.{key}',
                        f'Password policy {key} is {actual}, should be {expected}',
                        'IAM Password Policy',
                        f'Update IAM password policy to enable {key}'
                    )
        except ClientError as e:
            if e.response['Error']['Code'] == 'NoSuchEntity':
                self.add_finding(
                    'CRITICAL',
                    'CIS 1.5-1.11',
                    'No IAM password policy configured',
                    'IAM Password Policy',
                    'Create an IAM password policy meeting CIS requirements'
                )
    
    def check_root_access_keys(self):
        """CIS 1.4: Ensure no root account access keys exist"""
        try:
            summary = self.iam_client.get_account_summary()
            if summary['SummaryMap']['AccountAccessKeysPresent'] > 0:
                self.add_finding(
                    'CRITICAL',
                    'CIS 1.4',
                    'Root account has access keys',
                    'Root Account',
                    'Delete root account access keys immediately'
                )
        except ClientError as e:
            logger.error(f"Error checking root access keys: {e}")
    
    def check_iam_user_mfa(self):
        """CIS 1.2: Ensure MFA is enabled for all IAM users with console access"""
        try:
            paginator = self.iam_client.get_paginator('list_users')
            for page in paginator.paginate():
                for user in page['Users']:
                    username = user['UserName']
                    
                    # Check if user has console access
                    try:
                        self.iam_client.get_login_profile(UserName=username)
                        has_console = True
                    except ClientError as e:
                        if e.response['Error']['Code'] == 'NoSuchEntity':
                            has_console = False
                        else:
                            raise
                    
                    if has_console:
                        # Check MFA
                        mfa_devices = self.iam_client.list_mfa_devices(UserName=username)
                        if len(mfa_devices['MFADevices']) == 0:
                            self.add_finding(
                                'HIGH',
                                'CIS 1.2',
                                f'IAM user has console access without MFA',
                                username,
                                f'Enable MFA for user {username}'
                            )
        except ClientError as e:
            logger.error(f"Error checking IAM user MFA: {e}")
    
    def check_unused_credentials(self):
        """CIS 1.3: Ensure credentials unused for 90 days or greater are disabled"""
        try:
            report = self.iam_client.get_credential_report()
            import csv
            import io
            
            # Generate credential report if needed
            if report['ResponseMetadata']['HTTPStatusCode'] != 200:
                self.iam_client.generate_credential_report()
                return  # Will catch it on next run
            
            report_content = report['Content'].decode('utf-8')
            reader = csv.DictReader(io.StringIO(report_content))
            
            threshold = datetime.utcnow() - timedelta(days=90)
            
            for row in reader:
                username = row['user']
                if username == '<root_account>':
                    continue
                
                # Check password last used
                password_last_used = row.get('password_last_used')
                if password_last_used not in ['N/A', 'no_information']:
                    last_used = datetime.strptime(password_last_used, '%Y-%m-%dT%H:%M:%S+00:00')
                    if last_used < threshold:
                        self.add_finding(
                            'MEDIUM',
                            'CIS 1.3',
                            f'Password unused for more than 90 days',
                            username,
                            f'Disable or delete password for user {username}'
                        )
                
                # Check access keys
                for key_num in ['1', '2']:
                    key_last_used = row.get(f'access_key_{key_num}_last_used_date')
                    if key_last_used not in ['N/A', 'no_information']:
                        last_used = datetime.strptime(key_last_used, '%Y-%m-%dT%H:%M:%S+00:00')
                        if last_used < threshold:
                            self.add_finding(
                                'MEDIUM',
                                'CIS 1.3',
                                f'Access key {key_num} unused for more than 90 days',
                                username,
                                f'Deactivate or delete access key for user {username}'
                            )
        except ClientError as e:
            logger.error(f"Error checking unused credentials: {e}")
    
    def check_access_keys_rotation(self):
        """CIS 1.14: Ensure access keys are rotated every 90 days"""
        try:
            threshold = datetime.utcnow() - timedelta(days=90)
            
            paginator = self.iam_client.get_paginator('list_users')
            for page in paginator.paginate():
                for user in page['Users']:
                    username = user['UserName']
                    
                    keys = self.iam_client.list_access_keys(UserName=username)
                    for key in keys['AccessKeyMetadata']:
                        created = key['CreateDate'].replace(tzinfo=None)
                        if created < threshold:
                            self.add_finding(
                                'MEDIUM',
                                'CIS 1.14',
                                f'Access key older than 90 days',
                                f"{username}/{key['AccessKeyId']}",
                                f'Rotate access key for user {username}'
                            )
        except ClientError as e:
            logger.error(f"Error checking access key rotation: {e}")
    
    # Logging Checks (CIS 2.x)
    
    def check_cloudtrail_enabled(self):
        """CIS 2.1: Ensure CloudTrail is enabled in all regions"""
        try:
            trails = self.cloudtrail_client.describe_trails()['trailList']
            
            multi_region_trails = [t for t in trails if t.get('IsMultiRegionTrail', False)]
            
            if not multi_region_trails:
                self.add_finding(
                    'CRITICAL',
                    'CIS 2.1',
                    'No multi-region CloudTrail found',
                    'CloudTrail',
                    'Enable CloudTrail in all regions'
                )
            else:
                for trail in multi_region_trails:
                    status = self.cloudtrail_client.get_trail_status(Name=trail['TrailARN'])
                    if not status['IsLogging']:
                        self.add_finding(
                            'HIGH',
                            'CIS 2.1',
                            f'CloudTrail not logging',
                            trail['Name'],
                            f'Enable logging for CloudTrail {trail["Name"]}'
                        )
        except ClientError as e:
            logger.error(f"Error checking CloudTrail: {e}")
    
    def check_cloudtrail_log_validation(self):
        """CIS 2.2: Ensure CloudTrail log file validation is enabled"""
        try:
            trails = self.cloudtrail_client.describe_trails()['trailList']
            
            for trail in trails:
                if not trail.get('LogFileValidationEnabled', False):
                    self.add_finding(
                        'MEDIUM',
                        'CIS 2.2',
                        'CloudTrail log file validation not enabled',
                        trail['Name'],
                        f'Enable log file validation for trail {trail["Name"]}'
                    )
        except ClientError as e:
            logger.error(f"Error checking CloudTrail log validation: {e}")
    
    def check_cloudtrail_encryption(self):
        """CIS 2.7: Ensure CloudTrail logs are encrypted at rest using KMS"""
        try:
            trails = self.cloudtrail_client.describe_trails()['trailList']
            
            for trail in trails:
                if not trail.get('KmsKeyId'):
                    self.add_finding(
                        'HIGH',
                        'CIS 2.7',
                        'CloudTrail logs not encrypted with KMS',
                        trail['Name'],
                        f'Enable KMS encryption for trail {trail["Name"]}'
                    )
        except ClientError as e:
            logger.error(f"Error checking CloudTrail encryption: {e}")
    
    def check_cloudtrail_integration(self):
        """CIS 2.4: Ensure CloudTrail trails are integrated with CloudWatch Logs"""
        try:
            trails = self.cloudtrail_client.describe_trails()['trailList']
            
            for trail in trails:
                if not trail.get('CloudWatchLogsLogGroupArn'):
                    self.add_finding(
                        'MEDIUM',
                        'CIS 2.4',
                        'CloudTrail not integrated with CloudWatch Logs',
                        trail['Name'],
                        f'Integrate trail {trail["Name"]} with CloudWatch Logs'
                    )
        except ClientError as e:
            logger.error(f"Error checking CloudTrail CloudWatch integration: {e}")
    
    def check_s3_bucket_logging(self):
        """CIS 2.6: Ensure S3 bucket access logging is enabled on CloudTrail bucket"""
        try:
            trails = self.cloudtrail_client.describe_trails()['trailList']
            
            for trail in trails:
                bucket_name = trail['S3BucketName']
                
                try:
                    logging_config = self.s3_client.get_bucket_logging(Bucket=bucket_name)
                    if 'LoggingEnabled' not in logging_config:
                        self.add_finding(
                            'MEDIUM',
                            'CIS 2.6',
                            'S3 bucket access logging not enabled',
                            bucket_name,
                            f'Enable S3 access logging for bucket {bucket_name}'
                        )
                except ClientError as e:
                    if e.response['Error']['Code'] != 'NoSuchBucket':
                        raise
        except ClientError as e:
            logger.error(f"Error checking S3 bucket logging: {e}")
    
    def check_config_enabled(self):
        """CIS 2.5: Ensure AWS Config is enabled in all regions"""
        try:
            recorders = self.config_client.describe_configuration_recorders()
            recorder_status = self.config_client.describe_configuration_recorder_status()
            
            if not recorders['ConfigurationRecorders']:
                self.add_finding(
                    'HIGH',
                    'CIS 2.5',
                    'AWS Config not enabled',
                    'AWS Config',
                    'Enable AWS Config in all regions'
                )
            else:
                for status in recorder_status['ConfigurationRecordersStatus']:
                    if not status['recording']:
                        self.add_finding(
                            'HIGH',
                            'CIS 2.5',
                            'AWS Config recorder not recording',
                            status['name'],
                            f'Enable recording for Config recorder {status["name"]}'
                        )
        except ClientError as e:
            logger.error(f"Error checking AWS Config: {e}")
    
    # Monitoring Checks (CIS 3.x)
    
    def check_cloudwatch_alarms(self):
        """CIS 3.x: Check for required CloudWatch metric filters and alarms"""
        # This is a simplified check - in production, you'd verify specific alarms exist
        try:
            cloudwatch = boto3.client('cloudwatch', region_name=self.region)
            alarms = cloudwatch.describe_alarms()
            
            if len(alarms['MetricAlarms']) == 0:
                self.add_finding(
                    'MEDIUM',
                    'CIS 3.x',
                    'No CloudWatch alarms configured',
                    'CloudWatch',
                    'Create CloudWatch alarms for security monitoring per CIS 3.1-3.14'
                )
        except ClientError as e:
            logger.error(f"Error checking CloudWatch alarms: {e}")
    
    # Networking Checks (CIS 4.x)
    
    def check_security_groups(self):
        """CIS 4.1-4.3: Check security group rules"""
        try:
            security_groups = self.ec2_client.describe_security_groups()['SecurityGroups']
            
            for sg in security_groups:
                # Check for unrestricted SSH (CIS 4.1)
                for rule in sg.get('IpPermissions', []):
                    if rule.get('FromPort') == 22 or rule.get('ToPort') == 22:
                        for ip_range in rule.get('IpRanges', []):
                            if ip_range.get('CidrIp') == '0.0.0.0/0':
                                self.add_finding(
                                    'HIGH',
                                    'CIS 4.1',
                                    'Security group allows unrestricted SSH access',
                                    sg['GroupId'],
                                    f'Restrict SSH access in security group {sg["GroupId"]}'
                                )
                
                # Check for unrestricted RDP (CIS 4.2)
                for rule in sg.get('IpPermissions', []):
                    if rule.get('FromPort') == 3389 or rule.get('ToPort') == 3389:
                        for ip_range in rule.get('IpRanges', []):
                            if ip_range.get('CidrIp') == '0.0.0.0/0':
                                self.add_finding(
                                    'HIGH',
                                    'CIS 4.2',
                                    'Security group allows unrestricted RDP access',
                                    sg['GroupId'],
                                    f'Restrict RDP access in security group {sg["GroupId"]}'
                                )
        except ClientError as e:
            logger.error(f"Error checking security groups: {e}")
    
    def check_default_vpc(self):
        """CIS 4.4: Ensure default security group restricts all traffic"""
        try:
            vpcs = self.ec2_client.describe_vpcs()['Vpcs']
            
            for vpc in vpcs:
                # Get default security group
                sgs = self.ec2_client.describe_security_groups(
                    Filters=[
                        {'Name': 'vpc-id', 'Values': [vpc['VpcId']]},
                        {'Name': 'group-name', 'Values': ['default']}
                    ]
                )['SecurityGroups']
                
                for sg in sgs:
                    if sg.get('IpPermissions') or sg.get('IpPermissionsEgress'):
                        # Check if rules exist beyond the default deny-all
                        has_rules = any(
                            rule for rule in sg.get('IpPermissions', [])
                        ) or any(
                            rule for rule in sg.get('IpPermissionsEgress', [])
                            if not (len(rule.get('IpRanges', [])) == 1 and 
                                   rule['IpRanges'][0].get('CidrIp') == '0.0.0.0/0' and
                                   rule.get('IpProtocol') == '-1')
                        )
                        
                        if has_rules:
                            self.add_finding(
                                'MEDIUM',
                                'CIS 4.4',
                                'Default security group allows traffic',
                                sg['GroupId'],
                                f'Remove all rules from default security group {sg["GroupId"]}'
                            )
        except ClientError as e:
            logger.error(f"Error checking default VPC: {e}")
    
    def check_vpc_flow_logs(self):
        """CIS 3.9: Ensure VPC flow logging is enabled"""
        try:
            vpcs = self.ec2_client.describe_vpcs()['Vpcs']
            flow_logs = self.ec2_client.describe_flow_logs()['FlowLogs']
            
            vpc_with_logs = {fl['ResourceId'] for fl in flow_logs}
            
            for vpc in vpcs:
                if vpc['VpcId'] not in vpc_with_logs:
                    self.add_finding(
                        'MEDIUM',
                        'CIS 3.9',
                        'VPC does not have flow logs enabled',
                        vpc['VpcId'],
                        f'Enable VPC flow logs for VPC {vpc["VpcId"]}'
                    )
        except ClientError as e:
            logger.error(f"Error checking VPC flow logs: {e}")
    
    # Data Protection (CIS 5.x)
    
    def check_s3_bucket_encryption(self):
        """CIS 2.1.1: Ensure S3 buckets have encryption enabled"""
        try:
            buckets = self.s3_client.list_buckets()['Buckets']
            
            for bucket in buckets:
                bucket_name = bucket['Name']
                
                try:
                    encryption = self.s3_client.get_bucket_encryption(Bucket=bucket_name)
                except ClientError as e:
                    if e.response['Error']['Code'] == 'ServerSideEncryptionConfigurationNotFoundError':
                        self.add_finding(
                            'HIGH',
                            'CIS 2.1.1',
                            'S3 bucket does not have encryption enabled',
                            bucket_name,
                            f'Enable default encryption for bucket {bucket_name}'
                        )
        except ClientError as e:
            logger.error(f"Error checking S3 bucket encryption: {e}")
    
    def check_ebs_encryption(self):
        """CIS 2.2.1: Ensure EBS volume encryption is enabled"""
        try:
            volumes = self.ec2_client.describe_volumes()['Volumes']
            
            for volume in volumes:
                if not volume['Encrypted']:
                    self.add_finding(
                        'HIGH',
                        'CIS 2.2.1',
                        'EBS volume is not encrypted',
                        volume['VolumeId'],
                        f'Enable encryption for EBS volume {volume["VolumeId"]}'
                    )
        except ClientError as e:
            logger.error(f"Error checking EBS encryption: {e}")
    
    def check_rds_encryption(self):
        """CIS 2.3.1: Ensure RDS instances have encryption enabled"""
        try:
            rds_client = boto3.client('rds', region_name=self.region)
            instances = rds_client.describe_db_instances()['DBInstances']
            
            for instance in instances:
                if not instance.get('StorageEncrypted', False):
                    self.add_finding(
                        'HIGH',
                        'CIS 2.3.1',
                        'RDS instance does not have encryption enabled',
                        instance['DBInstanceIdentifier'],
                        f'Enable encryption for RDS instance {instance["DBInstanceIdentifier"]}'
                    )
        except ClientError as e:
            if e.response['Error']['Code'] != 'InvalidParameterValue':
                logger.error(f"Error checking RDS encryption: {e}")
    
    # Additional Security Services
    
    def check_guardduty_enabled(self):
        """Check if GuardDuty is enabled"""
        try:
            detectors = self.guardduty_client.list_detectors()['DetectorIds']
            
            if not detectors:
                self.add_finding(
                    'HIGH',
                    'GuardDuty',
                    'GuardDuty is not enabled',
                    'GuardDuty',
                    'Enable GuardDuty for threat detection'
                )
        except ClientError as e:
            logger.error(f"Error checking GuardDuty: {e}")
    
    def check_securityhub_enabled(self):
        """Check if Security Hub is enabled"""
        try:
            self.securityhub_client.describe_hub()
        except ClientError as e:
            if e.response['Error']['Code'] == 'InvalidAccessException':
                self.add_finding(
                    'MEDIUM',
                    'SecurityHub',
                    'Security Hub is not enabled',
                    'SecurityHub',
                    'Enable Security Hub for centralized security findings'
                )
    
    def remediate_findings(self, auto_remediate: bool = False):
        """Attempt to remediate findings (use with caution)"""
        if not auto_remediate:
            logger.info("Auto-remediation is disabled. Set auto_remediate=True to enable.")
            return
        
        logger.warning("Auto-remediation is enabled. This will make changes to your AWS account.")
        
        for finding in self.findings:
            # Implement safe, non-destructive remediations only
            if finding['control'] == 'CIS 2.1' and 'CloudTrail not logging' in finding['description']:
                try:
                    trail_name = finding['resource']
                    self.cloudtrail_client.start_logging(Name=trail_name)
                    logger.info(f"Remediated: Started logging for CloudTrail {trail_name}")
                except ClientError as e:
                    logger.error(f"Failed to remediate {finding['control']}: {e}")
    
    def send_sns_alert(self, sns_topic_arn: str):
        """Send findings to SNS topic"""
        try:
            critical_findings = [f for f in self.findings if f['severity'] == 'CRITICAL']
            high_findings = [f for f in self.findings if f['severity'] == 'HIGH']
            
            message = {
                'summary': {
                    'total_findings': len(self.findings),
                    'critical': len(critical_findings),
                    'high': len(high_findings)
                },
                'critical_findings': critical_findings[:10],  # Top 10
                'high_findings': high_findings[:10]
            }
            
            self.sns_client.publish(
                TopicArn=sns_topic_arn,
                Subject=f'AWS CIS Compliance Report - {len(self.findings)} issues found',
                Message=json.dumps(message, indent=2, default=str)
            )
            
            logger.info(f"Sent compliance report to SNS topic {sns_topic_arn}")
        except ClientError as e:
            logger.error(f"Error sending SNS alert: {e}")
    
    def generate_report(self, output_file: str = None) -> str:
        """Generate compliance report"""
        report = {
            'account_id': self.account_id,
            'region': self.region,
            'timestamp': datetime.utcnow().isoformat(),
            'summary': {
                'total_findings': len(self.findings),
                'by_severity': {
                    'CRITICAL': len([f for f in self.findings if f['severity'] == 'CRITICAL']),
                    'HIGH': len([f for f in self.findings if f['severity'] == 'HIGH']),
                    'MEDIUM': len([f for f in self.findings if f['severity'] == 'MEDIUM']),
                    'LOW': len([f for f in self.findings if f['severity'] == 'LOW'])
                }
            },
            'findings': self.findings
        }
        
        report_json = json.dumps(report, indent=2, default=str)
        
        if output_file:
            with open(output_file, 'w') as f:
                f.write(report_json)
            logger.info(f"Report written to {output_file}")
        
        return report_json


def lambda_handler(event, context):
    """Lambda function handler"""
    region = event.get('region', 'us-east-1')
    sns_topic_arn = event.get('sns_topic_arn')
    auto_remediate = event.get('auto_remediate', False)
    
    monitor = CISComplianceMonitor(region=region)
    monitor.run_all_checks()
    
    # Generate report
    report = monitor.generate_report()
    
    # Send SNS alert if configured
    if sns_topic_arn:
        monitor.send_sns_alert(sns_topic_arn)
    
    # Attempt remediation if enabled
    if auto_remediate:
        monitor.remediate_findings(auto_remediate=True)
    
    return {
        'statusCode': 200,
        'body': report
    }


if __name__ == '__main__':
    # For standalone execution
    import argparse
    
    parser = argparse.ArgumentParser(description='AWS CIS Compliance Monitor')
    parser.add_argument('--region', default='us-east-1', help='AWS region')
    parser.add_argument('--output', help='Output file for report')
    parser.add_argument('--sns-topic', help='SNS topic ARN for alerts')
    parser.add_argument('--auto-remediate', action='store_true', 
                       help='Attempt automatic remediation (use with caution)')
    
    args = parser.parse_args()
    
    monitor = CISComplianceMonitor(region=args.region)
    monitor.run_all_checks()
    
    # Generate and save report
    report = monitor.generate_report(output_file=args.output)
    print(report)
    
    # Send SNS alert if configured
    if args.sns_topic:
        monitor.send_sns_alert(args.sns_topic)
    
    # Attempt remediation if enabled
    if args.auto_remediate:
        monitor.remediate_findings(auto_remediate=True)
```

### Lambda Deployment Package (`scripts/deploy_lambda.sh`)

```bash
#!/bin/bash
# Deploy CIS Compliance Monitor as Lambda function

set -e

LAMBDA_FUNCTION_NAME="cis-compliance-monitor"
LAMBDA_ROLE_ARN="arn:aws:iam::ACCOUNT_ID:role/lambda-cis-monitor-role"  # Update this
SNS_TOPIC_ARN="arn:aws:sns:us-east-1:ACCOUNT_ID:security-alerts"  # Update this

echo "Creating Lambda deployment package..."

# Create deployment directory
rm -rf lambda-package
mkdir lambda-package
cd lambda-package

# Copy script
cp ../cis_compliance_monitor.py .

# Create requirements.txt
cat > requirements.txt << EOF
boto3
botocore
EOF

# Install dependencies
pip install -r requirements.txt -t .

# Create zip package
zip -r ../cis-compliance-monitor.zip .

cd ..

echo "Deployment package created: cis-compliance-monitor.zip"

# Create or update Lambda function
aws lambda create-function \
    --function-name ${LAMBDA_FUNCTION_NAME} \
    --runtime python3.9 \
    --role ${LAMBDA_ROLE_ARN} \
    --handler cis_compliance_monitor.lambda_handler \
    --timeout 900 \
    --memory-size 512 \
    --zip-file fileb://cis-compliance-monitor.zip \
    --environment Variables="{SNS_TOPIC_ARN=${SNS_TOPIC_ARN}}" \
    || aws lambda update-function-code \
    --function-name ${LAMBDA_FUNCTION_NAME} \
    --zip-file fileb://cis-compliance-monitor.zip

echo "Lambda function deployed successfully"

# Create EventBridge rule to run daily
aws events put-rule \
    --name daily-cis-compliance-check \
    --schedule-expression "rate(1 day)"

aws events put-targets \
    --rule daily-cis-compliance-check \
    --targets "Id"="1","Arn"="arn:aws:lambda:us-east-1:ACCOUNT_ID:function:${LAMBDA_FUNCTION_NAME}"

echo "Scheduled daily execution configured"
```

---

## Testing & Validation

### Step-by-Step Testing Guide

**1. Validate Terraform Configuration**
```bash
cd environments/prod
terraform fmt -recursive
terraform validate
terraform plan
```

**2. Test Python Script Locally**
```bash
cd scripts
python3 cis_compliance_monitor.py --region us-east-1 --output compliance-report.json
```

**3. Review Compliance Report**
```bash
cat compliance-report.json | jq '.summary'
```

**4. Deploy Infrastructure**
```bash
cd environments/prod
terraform apply -auto-approve
```

**5. Verify AWS Services**
```bash
# Verify CloudTrail
aws cloudtrail describe-trails --region us-east-1

# Verify GuardDuty
aws guardduty list-detectors --region us-east-1

# Verify Security Hub
aws securityhub describe-hub --region us-east-1

# Verify Config
aws configservice describe-configuration-recorders --region us-east-1
```

**6. Test Lambda Function**
```bash
aws lambda invoke \
    --function-name cis-compliance-monitor \
    --payload '{"region": "us-east-1"}' \
    response.json

cat response.json | jq '.'
```

---

## Deployment Guide

### Complete Deployment Workflow

**1. Prepare Environment**
```bash
# Clone or create project structure
mkdir -p aws-security-baseline
cd aws-security-baseline

# Set up directory structure (as shown in Phase 1)
```

**2. Configure Terraform Backend**
```bash
# Create S3 bucket for Terraform state
aws s3api create-bucket \
    --bucket your-terraform-state-bucket \
    --region us-east-1

# Enable versioning
aws s3api put-bucket-versioning \
    --bucket your-terraform-state-bucket \
    --versioning-configuration Status=Enabled

# Create DynamoDB table for state locking
aws dynamodb create-table \
    --table-name terraform-state-lock \
    --attribute-definitions AttributeName=LockID,AttributeType=S \
    --key-schema AttributeName=LockID,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
```

**3. Update Configuration Files**
```bash
# Edit terraform.tfvars with your values
vi environments/prod/terraform.tfvars
```

**4. Initialize and Deploy**
```bash
cd environments/prod

# Initialize Terraform
terraform init

# Review execution plan
terraform plan -out=tfplan

# Apply configuration
terraform apply tfplan

# Save outputs
terraform output > terraform-outputs.txt
```

**5. Deploy Monitoring Script**
```bash
cd ../../scripts

# Update Lambda deployment script with actual ARNs
vi deploy_lambda.sh

# Deploy Lambda function
chmod +x deploy_lambda.sh
./deploy_lambda.sh
```

**6. Verify Deployment**
```bash
# Run compliance check
python3 cis_compliance_monitor.py \
    --region us-east-1 \
    --output initial-compliance-report.json \
    --sns-topic arn:aws:sns:us-east-1:ACCOUNT_ID:security-alerts
```

## Services Utilized

### Complete Service Matrix

| Service | Purpose | CIS Control |
|---------|---------|-------------|
| **IAM** | Identity and access management, password policies, MFA | 1.x |
| **VPC** | Network isolation, subnets, security groups | 4.x |
| **EC2** | Security groups, EBS encryption | 4.x |
| **S3** | Encrypted storage for logs, versioning, lifecycle policies | 2.x, 5.x |
| **KMS** | Encryption key management, key rotation | 2.7, 3.7 |
| **CloudTrail** | Audit logging of API calls across all regions | 2.1-2.7 |
| **AWS Config** | Configuration change tracking and compliance | 2.5 |
| **GuardDuty** | Intelligent threat detection | N/A |
| **Security Hub** | Centralized security findings and compliance dashboard | N/A |
| **CloudWatch** | Logs aggregation, metric filters, alarms | 3.x |
| **SNS** | Security alert notifications | 3.x |
| **Lambda** | Automated compliance monitoring execution | N/A |
| **EventBridge** | Scheduled Lambda execution | N/A |

