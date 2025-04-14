# Day 5 – Auto Scaling with Metrics
**Project Title**: Dynamic EC2 Scaling with ALB & CloudWatch Metrics  
**Date**: 14-Apr-2025

## Objective
Automatically scale EC2 instances in and out based on CPU usage. Use an Application Load Balancer to distribute traffic and simulate load to observe Auto Scaling in action.

---

## VPC & Network Setup
- Used existing **custom VPC**
- At least 2 **public subnets** (for ALB and ASG instances)
- **Internet Gateway** attached
- **Route Tables** updated for internet access

---

## Launch Template
- **AMI**: Amazon Linux 2
- **User Data**: (Nginx installation + test page)
- **Security Group**: HTTP (80), SSH (22 – optional for testing)
- **Key Pair**: For optional SSH login

---

## Application Load Balancer (ALB)
- Created ALB in **two public subnets**
- **Target Group** with health check on port **80**
- **Listener**: HTTP → Forward to Target Group

---

## Auto Scaling Group (ASG)
- Min: 1, Desired: 1, Max: 10
- **Launch Template** used
- **Attached to ALB**
- **Scaling Policy – Step Scaling**:
  - **Scale Out**:
    - If CPU > 70% → Add 1 instance
    - If CPU > 90% → Add 2 instances
  - **Scale In**:
    - If CPU < 20% → Remove 1 instance

---

## Load Testing
- SSH into instance:
  ```bash
  sudo yum install stress -y
  stress --cpu 2 --timeout 300
