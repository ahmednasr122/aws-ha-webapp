# AWS High Availability Web Application

A highly available web application on AWS using Application Load Balancer and Auto Scaling Group.

## 🏗️ Architecture

- **VPC** with public and private subnets in 2 Availability Zones
- **Application Load Balancer** for traffic distribution
- **Auto Scaling Group** with 2-4 EC2 instances
- **Private subnets** - EC2 instances have no public IPs
- **NAT Gateway** for outbound internet access

## 🚀 Quick Start

###  AWS Console (Manual)
See [DEPLOYMENT.md](DEPLOYMENT.md) for step-by-step instructions.

```

## 📊 Features

- ✅ High Availability across 2 AZs
- ✅ Auto Scaling based on CPU (70% threshold)
- ✅ Load Balancing with health checks
- ✅ Enhanced security (private subnets)
- ✅ Cost optimized (t3.micro instances)

## 💰 Estimated Cost

~$60-70/month in us-east-1:
- EC2 instances (2x t3.micro): ~$12/month
- Application Load Balancer: ~$16/month
- NAT Gateway: ~$32/month

## 🧹 Cleanup

Delete in this order:
1. Auto Scaling Group
2. Load Balancer
3. Target Group
4. Launch Template
5. NAT Gateway
6. VPC
