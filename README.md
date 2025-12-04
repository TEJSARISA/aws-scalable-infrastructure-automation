# AWS Scalable Cloud Infrastructure Automation

A comprehensive Infrastructure-as-Code (IaC) project demonstrating automated deployment of highly available, scalable cloud infrastructure on AWS using EC2, IAM, Security Groups, and DevOps best practices.

## Project Overview

This project showcases enterprise-grade cloud infrastructure automation principles by deploying a multi-tier, highly available architecture across multiple AWS availability zones. It demonstrates:

- **Automated EC2 Provisioning**: User-data bootstrap scripts for automatic web server installation
- **High Availability Design**: Multi-subnet deployment across multiple AZs
- **Secure Access Control**: IAM roles with least-privilege policies and instance profiles
- **Network Security**: Custom Security Groups with restrictive inbound rules
- **Infrastructure-as-Code**: Fully scripted, reproducible infrastructure
- **DevOps Best Practices**: Version control, documentation, and automation

## Project Structure

```
aws-scalable-infrastructure-automation/
├── README.md                          # This file
├── ARCHITECTURE.md                    # Detailed architecture documentation
├── infrastructure/
│   ├── vpc-setup.sh                  # VPC and subnet configuration
│   ├── ec2-instances.sh              # EC2 instance deployment
│   ├── iam-roles.tf                  # IAM roles and instance profiles
│   ├── security-groups.sh            # Security group rules
│   └── terraform/
│       ├── main.tf                   # Terraform main configuration
│       ├── variables.tf              # Variable definitions
│       ├── outputs.tf                # Output definitions
│       └── terraform.tfvars          # Variable values
├── scripts/
│   ├── bootstrap.sh                  # User-data bootstrap script for EC2
│   ├── deploy.sh                     # Automated deployment script
│   ├── validate.sh                   # Infrastructure validation script
│   └── cleanup.sh                    # Resource cleanup script
├── docs/
│   ├── DEPLOYMENT_GUIDE.md           # Step-by-step deployment instructions
│   ├── SECURITY.md                   # Security implementation details
│   └── TROUBLESHOOTING.md            # Common issues and solutions
├── config/
│   ├── vpc-config.json               # VPC configuration
│   ├── ec2-config.json               # EC2 instance configuration
│   └── iam-policy.json               # IAM policy definition
└── .gitignore                        # Git ignore file
```

## Key Features

### 1. Automated EC2 Provisioning
- User-data scripts execute automatically on instance launch
- Apache web server installation and configuration
- Application deployment automation
- Health check setup

### 2. High-Availability Architecture
- Multi-subnet deployment (Public + Private subnets)
- Instances distributed across multiple availability zones
- Load balancing ready design
- Auto-scaling group support

### 3. Security Implementation
- **IAM Instance Profile**: Enables secure S3 access without embedded credentials
- **Least-Privilege Policies**: Restricted permissions for resources
- **Security Groups**: 
  - SSH (port 22) - restricted to admin IPs
  - HTTP (port 80) - open to all
  - HTTPS (port 443) - open to all
  - Egress traffic: unrestricted (customizable)

### 4. Infrastructure-as-Code
- Terraform configuration for reproducibility
- AWS CLI scripts for alternative deployment
- Parameterized configurations
- Environment-specific variable files

## Prerequisites

- AWS Account with appropriate permissions
- AWS CLI v2 configured with credentials
- Terraform v1.0+ (optional, for Terraform deployment)
- Bash shell environment
- SSH key pair created in your AWS region
- Python 3.6+ (for validation scripts)

## AWS Credentials Setup

### Using AWS CLI

```bash
# Configure AWS credentials
aws configure

# When prompted, enter:
# AWS Access Key ID: [your-access-key]
# AWS Secret Access Key: [your-secret-key]
# Default region name: us-east-1
# Default output format: json
```

### Verify Configuration

```bash
aws sts get-caller-identity
```

## Quick Start

### Option 1: Using Automated Deployment Script

```bash
# Clone the repository
git clone https://github.com/TEJSARISA/aws-scalable-infrastructure-automation.git
cd aws-scalable-infrastructure-automation

# Make scripts executable
chmod +x scripts/*.sh

# Configure variables
cp config/example.env .env
# Edit .env with your settings

# Run deployment
./scripts/deploy.sh
```

### Option 2: Using Terraform

```bash
cd infrastructure/terraform

# Initialize Terraform
terraform init

# Review planned changes
terraform plan

# Deploy infrastructure
terraform apply

# Retrieve outputs
terraform output
```

### Option 3: Using AWS CLI

```bash
# Create VPC
bash infrastructure/vpc-setup.sh

# Create Security Groups
bash infrastructure/security-groups.sh

# Create IAM roles
bash infrastructure/iam-roles.tf

# Deploy EC2 instances
bash infrastructure/ec2-instances.sh
```

## Configuration

### Environment Variables

Create a `.env` file in the project root:

```bash
# AWS Configuration
AWS_REGION=us-east-1
AWS_PROFILE=default

# VPC Configuration
VPC_CIDR=10.0.0.0/16
PUBLIC_SUBNET_CIDR=10.0.1.0/24
PRIVATE_SUBNET_CIDR=10.0.2.0/24

# EC2 Configuration
INSTANCE_TYPE=t3.micro
KEY_PAIR_NAME=my-key-pair
INSTANCE_COUNT=2
AMI_ID=ami-0c55b159cbfafe1f0  # Amazon Linux 2 AMI

# Project Tags
PROJECT_NAME=aws-infrastructure
ENVIRONMENT=production
OWNER=terraform
```

## Deployment Validation

After deployment, validate the infrastructure:

```bash
# Check EC2 instances
aws ec2 describe-instances --filters "Name=tag:Project,Values=aws-infrastructure"

# Verify Security Groups
aws ec2 describe-security-groups --filters "Name=tag:Project,Values=aws-infrastructure"

# Check IAM roles
aws iam list-roles --query "Roles[?contains(RoleName, 'infrastructure')]"

# Test web server connectivity
curl http://<instance-public-ip>
```

## Cost Optimization

- **t3.micro instances**: Free tier eligible
- **Estimated monthly cost**: $5-15 USD (depending on usage)
- **Free Tier eligible**: VPC, Security Groups, IAM roles (no charges)

### Stop Resources (Development)

```bash
# Stop instances to reduce costs
aws ec2 stop-instances --instance-ids i-xxxxx

# Restart instances
aws ec2 start-instances --instance-ids i-xxxxx
```

## Security Best Practices Implemented

1. **IAM Least Privilege**: Instance roles have minimal required permissions
2. **Network Isolation**: Private subnets for sensitive resources
3. **Security Groups**: Restrictive inbound rules, unrestricted egress
4. **No Hardcoded Credentials**: All credentials through IAM roles/profiles
5. **Encryption Ready**: VPC Flow Logs and CloudTrail integration support
6. **SSH Key-based Auth**: No password authentication enabled

## Troubleshooting

### Instance Not Reachable

```bash
# Check instance status
aws ec2 describe-instance-status --instance-ids i-xxxxx

# Check security group rules
aws ec2 describe-security-groups --group-ids sg-xxxxx

# Verify SSH key permissions
chmod 400 ~/.ssh/key-pair.pem

# SSH with debugging
ssh -vvv -i ~/.ssh/key-pair.pem ec2-user@<public-ip>
```

### User-Data Script Not Executing

```bash
# Check cloud-init logs
sudo tail -100 /var/log/cloud-init-output.log
sudo tail -100 /var/log/cloud-init.log

# Verify web server status
sudo systemctl status httpd
sudo systemctl start httpd
```

## Cleanup

### Using Cleanup Script

```bash
./scripts/cleanup.sh
```

### Using Terraform

```bash
cd infrastructure/terraform
terraform destroy
```

### Manual AWS CLI Cleanup

```bash
# Terminate instances
aws ec2 terminate-instances --instance-ids i-xxxxx

# Delete security groups
aws ec2 delete-security-group --group-id sg-xxxxx

# Delete VPC (after instances and subnets are removed)
aws ec2 delete-vpc --vpc-id vpc-xxxxx

# Delete IAM role
aws iam delete-role --role-name infrastructure-instance-role
```

## Learning Outcomes

After completing this project, you will understand:

✅ AWS VPC architecture and networking
✅ EC2 instance provisioning and configuration
✅ IAM roles and instance profiles
✅ Security Groups and network access control
✅ User-data scripts for automation
✅ Infrastructure-as-Code principles
✅ AWS CLI usage and automation
✅ High-availability architecture patterns
✅ DevOps best practices
✅ Cost optimization strategies

## Advanced Topics

- Auto Scaling Groups for elasticity
- Application Load Balancer (ALB) setup
- RDS database integration
- S3 bucket configuration for application data
- CloudWatch monitoring and alarms
- VPC Peering and Site-to-Site VPN
- AWS Systems Manager integration
- Infrastructure testing and validation

## References

- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [EC2 User Guide](https://docs.aws.amazon.com/ec2/)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest)
- [AWS CLI Documentation](https://docs.aws.amazon.com/cli/)

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

MIT License - See LICENSE file for details

## Support

For issues, questions, or suggestions:
- Open an issue on GitHub
- Check TROUBLESHOOTING.md
- Review ARCHITECTURE.md for detailed design information

## Author

TEJSARISA - Cloud Infrastructure & DevOps

---

**Last Updated**: December 4, 2025
**Project Status**: Active Development
**Version**: 1.0.0
