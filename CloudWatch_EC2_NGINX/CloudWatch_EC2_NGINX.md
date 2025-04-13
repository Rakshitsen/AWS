## Day 5 - Adding CloudWatch to Day 4 Architecture

On Day 5, we enhanced our existing Day 4 architecture by integrating **Amazon CloudWatch**. The setup includes a **private EC2 instance** running **NGINX** with a custom index file.

### Background
Yesterday, after completing the hands-on work, I stopped the EC2 instance to save free-tier usage. I also deleted the NAT Gateway and released its Elastic IP.

Today, I recreated the following:
- A new **NAT Gateway**
- Allocated a fresh **Elastic IP**
- Updated the **private subnet route table** so that the private instance can access the internet while staying hidden.
- Relaunched the **Bastion Host** and the **Private Instance**

---

### Step-by-Step Configuration

1. Ping Test
Check internet connectivity from the private instance:

ping google.com


2. Install CloudWatch Agent
On Amazon Linux (YUM-based), run:
sudo yum install amazon-cloudwatch-agent -y

3. Create CloudWatch Agent Configuration
Create a configuration file to monitor CPU, memory, and NGINX logs:
sudo nano /opt/aws/amazon-cloudwatch-agent/bin/cloudwatch-config.json


4. Start CloudWatch Agent
Start the agent with your custom config file:

sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
-a fetch-config -m ec2 \
-c file:/opt/aws/amazon-cloudwatch-agent/bin/cloudwatch-config.json -s




Troubleshooting: No Logs in CloudWatch
At first, no logs appeared in CloudWatch. After investigating, I realized the instance didn't have the right IAM role permissions to push logs.

First Attempt (Incorrect Policy)
Attached IAM Policy: CloudWatchEventsFullAccess

Result: No logs appeared.

Correct Policy
Attached IAM Policy: CloudWatchAgentServerPolicy

Result: Logs successfully appeared in CloudWatch.



IAM Policy Explanation
CloudWatchEventsFullAccess
Used for CloudWatch Events (EventBridge)

Enables event triggers, schedules, and automation

Does not allow pushing logs from EC2 to CloudWatch

CloudWatchAgentServerPolicy
Required to send logs/metrics from EC2 to CloudWatch

Works with CloudWatch Agent

This is the correct policy for pushing logs to CloudWatch


