Project: Private EC2 Hosting Nginx Website with S3 Sync & ALB using Bastion Host (with Cron Automation)
Step-by-Step Breakdown
1. VPC and Subnet Setup
Created a new VPC.

Created three subnets:

Two public subnets (required for the Application Load Balancer).

One private subnet (for hosting the web server).

2. Bastion Host Setup
Launched an Amazon Linux EC2 instance (Bastion Host) in one of the public subnets.

In the Bastion Host Security Group, added inbound SSH (port 22) access from my IP only for secure access.

3. Private Instance Setup
Launched a private EC2 instance (Amazon Linux) in the private subnet.

Tried to SSH into it via Bastion, but realized the private key (.pem) wasn’t available inside the Bastion Host.

Used the scp command from my local machine to copy the private key into the Bastion Host securely.

Successfully SSH’d from Bastion Host into the private EC2 instance.

4. NAT Gateway Setup (for Internet in Private Subnet)
Couldn’t run yum update or install packages in the private instance — no internet access.

Created a NAT Gateway and placed it in one of the public subnets.

Updated private subnet route table to use the NAT Gateway for outbound internet.

Now I could install and update packages in the private instance.

5. Security Group Adjustments
In the private instance’s SG, allowed:

SSH access only from Bastion Host's SG.

HTTP and HTTPS access from ALB (to be added later).

6. Nginx Installation
Installed, started, and enabled Nginx on the private EC2 instance.

Created an IAM Role with AmazonS3ReadOnlyAccess, and attached it to the private EC2 instance.

Used the aws s3 sync command to sync static files (e.g., index.html) from S3 into /usr/share/nginx/html.

7. Verification
Used curl -I localhost to verify Nginx is serving the page (HTTP/1.1 200 OK).

8. Load Balancer Setup
Created a Target Group, added the private instance as a target.

Created an Application Load Balancer (ALB) in the two public subnets.

Attached the Target Group to the ALB.

Opened the ALB DNS name in browser — and saw my hosted website served from the private instance.

9. Cron Job for Continuous Sync
Installed cron (cronie package).

Started and enabled the crond service.

Set up a root-level cron job to sync S3 to Nginx HTML folder every 2 minutes:


*/2 * * * * /usr/bin/aws s3 sync s3://my-nginx-content-bucket-2025 /usr/share/nginx/html --region us-east-2 >> /home/ec2-user/sync.log 2>&1
This keeps the website updated automatically whenever a new index.html file is uploaded to S3.



# S3 to Nginx Auto-Sync via Cron on AWS EC2 (Private Instance)

## Overview
This guide explains how to keep your private EC2 instance's Nginx web server in sync with an S3 bucket using AWS CLI and a cron job.

---

## Prerequisites

- Private EC2 instance (Amazon Linux)
- IAM role attached to instance with S3 read permissions
- AWS CLI pre-installed (default in Amazon Linux)
- S3 bucket (e.g., `my-nginx-content-bucket-2025`) with `index.html` or site content
- Bastion Host for SSH access
- ALB configured to point to private EC2 (optional)

---

## Step-by-Step Setup

### 1. Connect to Private EC2 (via Bastion)
```bash
scp -i bastion-key.pem my-key.pem ec2-user@<Bastion-IP>:/home/ec2-user/
ssh -i bastion-key.pem ec2-user@<Bastion-IP>
ssh -i my-key.pem ec2-user@<Private-EC2-IP>



# 1. Install Cron Service (If not already installed)
sudo yum install cronie -y

# 2. Start and Enable Cron Service
sudo systemctl start crond
sudo systemctl enable crond

# 3. Edit Root's Crontab
sudo crontab -e

*/2 * * * * /usr/bin/aws s3 sync s3://my-nginx-content-bucket-2025 /usr/share/nginx/html --region us-east-2 >> /home/ec2-user/sync.log 2>&1



Purpose of Each Line
*/2 * * * * → Run every 2 minutes

/usr/bin/aws → Full path to AWS CLI (check with which aws)

>> /home/ec2-user/sync.log 2>&1 → Save logs & errors for debuggingPurpose of Each Line
*/2 * * * * → Run every 2 minutes

/usr/bin/aws → Full path to AWS CLI (check with which aws)

>> /home/ec2-user/sync.log 2>&1 → Save logs & errors for debugging


cat /home/ec2-user/sync.log


