# 🗂️ Day 4 — RDS, DynamoDB & AWS Lambda

> **7 Days of AWS Challenge** | Day 4 of 7

![AWS](https://img.shields.io/badge/AWS-RDS-orange?style=for-the-badge&logo=amazon-aws)
![DynamoDB](https://img.shields.io/badge/AWS-DynamoDB-yellow?style=for-the-badge&logo=amazon-aws)
![Lambda](https://img.shields.io/badge/AWS-Lambda-green?style=for-the-badge&logo=amazon-aws)
![EC2](https://img.shields.io/badge/AWS-EC2-blue?style=for-the-badge&logo=amazon-aws)
![Status](https://img.shields.io/badge/Status-Completed-green?style=for-the-badge)

---

## 📌 Project Overview

In this project, I learned how to set up a managed relational database on Amazon RDS, deploy a two-tier Flask application connected to RDS, and automate EC2 instance start/stop using AWS Lambda and EventBridge — saving costs during non-business hours.

---

## 🏗️ Architecture

```
Internet
    │
    ▼
Elastic Load Balancer
    │
    ▼
Auto Scaling Group
    └── EC2 Instance (Flask App)
            │
            ▼
    Amazon RDS (MySQL 8.0)
    └── Private Subnet (No Public Access)
            └── Security Group (Port 3306 — EC2 SG only)

EventBridge (Cron Schedule)
    │
    ▼
AWS Lambda (ec2-auto-stop / ec2-auto-start)
    └── Boto3 → EC2 Instances (AutoStop=true)
        └── IAM Role (EC2StartStop + CloudWatch Logs)
```

---

## ✅ What I Completed

- [x] Created MySQL RDS instance (db.t3.micro, Free Tier)
- [x] Configured RDS in private subnet — no public access
- [x] Set RDS security group to allow port 3306 from EC2 SG only
- [x] Connected EC2 to RDS via MySQL client
- [x] Performed CRUD operations on RDS from EC2
- [x] Cloned and configured two-tier Flask app on EC2
- [x] Connected Flask app to RDS using environment variables
- [x] Deployed Flask app behind an Elastic Load Balancer
- [x] Set up Auto Scaling Group (min=1, max=3)
- [x] Created IAM role for Lambda with EC2 + CloudWatch permissions
- [x] Wrote Lambda function (Python Boto3) to start/stop EC2 by tag
- [x] Tagged EC2 instances with `AutoStop=true`
- [x] Created EventBridge rules for 9 PM stop and 9 AM start (IST)
- [x] Tested Lambda manually — verified instances stopped/started

---

## 🔧 Tech Stack

| Service | Purpose |
|---|---|
| Amazon RDS (MySQL 8.0) | Managed relational database |
| AWS Lambda (Python 3.12) | Serverless EC2 automation |
| Amazon EventBridge | Cron-based Lambda scheduler |
| AWS EC2 (t3.micro) | Flask app hosting |
| Elastic Load Balancer | Traffic distribution |
| Auto Scaling Group | EC2 scaling |
| IAM Role | Lambda permissions |
| Python Boto3 | AWS SDK for Lambda |

---

## 🛡️ IAM Policy for Lambda (EC2StartStopPolicy)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EC2StartStop",
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:DescribeInstances",
        "ec2:DescribeInstanceStatus"
      ],
      "Resource": "*"
    },
    {
      "Sid": "CloudWatchLogs",
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    }
  ]
}
```

---

## 🌀 Lambda Function (ec2-auto-stop / ec2-auto-start)

```python
import boto3
import os
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    action = os.environ.get("ACTION", "stop").lower()
    region = os.environ.get("AWS_REGION", "ap-south-1")

    ec2 = boto3.client("ec2", region_name=region)

    response = ec2.describe_instances(
        Filters=[
            {"Name": "tag:AutoStop", "Values": ["true"]},
            {"Name": "instance-state-name",
             "Values": ["running"] if action == "stop" else ["stopped"]}
        ]
    )

    instance_ids = [
        i["InstanceId"]
        for r in response["Reservations"]
        for i in r["Instances"]
    ]

    if not instance_ids:
        logger.info(f"No instances to {action}.")
        return {"status": "nothing to do", "instances": []}

    if action == "stop":
        ec2.stop_instances(InstanceIds=instance_ids)
        logger.info(f"Stopped: {instance_ids}")
    else:
        ec2.start_instances(InstanceIds=instance_ids)
        logger.info(f"Started: {instance_ids}")

    return {"action": action, "instances": instance_ids, "count": len(instance_ids)}
```

---

## 🖥️ CLI & Commands Used

### RDS — Connect from EC2
```bash
# Install MySQL client
sudo apt update && sudo apt install mysql-client -y

# Connect to RDS
mysql -h <rds-endpoint>.rds.amazonaws.com -u admin -p

# CRUD test
CREATE DATABASE ecommerce;
USE ecommerce;

CREATE TABLE products (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  price DECIMAL(10,2),
  stock INT
);

INSERT INTO products (name, price, stock)
VALUES ('Laptop', 49999.00, 10), ('Mouse', 499.00, 100);

SELECT * FROM products;
```

### Flask App — Environment Variables
```bash
# Set env vars on EC2
export DB_HOST="<rds-endpoint>.rds.amazonaws.com"
export DB_USER="admin"
export DB_PASS="your-password"
export DB_NAME="ecommerce"
export DB_PORT="3306"

# Clone and run the app
git clone https://github.com/LondheShubham153/two-tier-flask-app
cd two-tier-flask-app
pip install flask pymysql
python app.py
```

### EventBridge Cron Schedules
```bash
# Stop at 9 PM IST (3:30 PM UTC) — weekdays
cron(30 15 ? * MON-FRI *)

# Start at 9 AM IST (3:30 AM UTC) — weekdays
cron(30 3 ? * MON-FRI *)
```

### Lambda — Tag EC2 for AutoStop
```bash
# Tag an instance via CLI
aws ec2 create-tags \
  --resources i-XXXXXXXXXXXXXXXXX \
  --tags Key=AutoStop,Value=true

# Test Lambda manually via CLI
aws lambda invoke \
  --function-name ec2-auto-stop \
  --payload '{}' \
  response.json

cat response.json
```

---

## 📸 Screenshots

### Task 1 — RDS Setup

#### RDS MySQL Instance — Available ✅
![RDS Available](screenshots/rds-available.png)

#### RDS Security Group — Port 3306 from EC2 SG Only ✅
![RDS Security Group](screenshots/rds-security-group.png)

#### EC2 Connected to RDS via MySQL Client ✅
![RDS Connection](screenshots/rds-connection.png)

#### CRUD Operations on RDS ✅
![RDS CRUD](screenshots/rds-crud.png)

---

### Task 2 — Two-Tier Flask App

#### Flask App Running on EC2 ✅
![Flask Running](screenshots/flask-running.png)

#### Load Balancer DNS Returning Flask Response ✅
![Load Balancer](screenshots/load-balancer.png)

---

### Task 3 — Lambda Automation

#### Lambda Function Created (ec2-auto-stop) ✅
![Lambda Function](screenshots/lambda-function.png)

#### Lambda Test — EC2 Instance Stopped ✅
![Lambda Test](screenshots/lambda-test.png)

#### EventBridge Rule — Stop Schedule Created ✅
![EventBridge Rule](screenshots/eventbridge-rule.png)

#### CloudWatch Logs — Stopped Instance ID Confirmed ✅
![CloudWatch Logs](screenshots/cloudwatch-logs.png)

---

## 💡 Key Learnings

- **RDS vs self-managed DB** — RDS handles backups, patching, and failover automatically
- **Never open port 3306 to 0.0.0.0/0** — always restrict to EC2 security group ID only
- **Environment variables** keep database credentials out of source code and Git
- **Lambda + EventBridge** = powerful cost-saving automation with zero servers
- **AutoStop tag pattern** means Lambda never needs updating when instances change
- **EventBridge always uses UTC** — remember to convert IST (UTC+5:30) before setting cron
- **Boto3 state filter** prevents errors from stopping already-stopped instances
- **Multi-AZ RDS** provides automatic failover — no manual intervention needed
- **db.t3.micro is free tier** in ap-south-1 — always stop RDS when not in use
- **CloudWatch Logs** are essential for debugging Lambda — check them first on failures

---

## 🔐 Security Best Practices Followed

- RDS placed in private subnet — no public access enabled
- RDS security group allows only EC2 security group as source (not IP or 0.0.0.0/0)
- Database credentials stored as environment variables — never hardcoded
- Lambda IAM role follows Principle of Least Privilege
- EC2 instances tagged for targeted automation — not blanket stop/start
- `.env` file added to `.gitignore` to prevent credential leaks

---

## 💰 Cost Breakdown

| Resource | Cost |
|---|---|
| RDS db.t3.micro (Free Tier) | Free tier |
| Lambda (1M requests/month free) | Free tier |
| EventBridge rules | Free tier |
| EC2 t3.micro (stopped after testing) | Free tier |
| Elastic Load Balancer | ~$0.02/hr if left running |
| **Total** | **~$0** |

> ⚠️ Stop RDS and delete the Load Balancer after testing — both charge even when idle.

---

## 🚀 How to Reproduce

1. Create MySQL RDS instance (db.t3.micro, Free Tier, no public access)
2. Set RDS security group inbound: port 3306, source = EC2 security group ID
3. Connect from EC2 using `mysql -h <endpoint> -u admin -p`
4. Clone `two-tier-flask-app`, set DB env vars, run Flask app
5. Create IAM role for Lambda with EC2 + CloudWatch permissions
6. Write Lambda function with `ACTION` env var for stop/start
7. Tag EC2 instances with `AutoStop=true`
8. Create two EventBridge rules — stop (cron `30 15 ? * MON-FRI *`) and start (cron `30 3 ? * MON-FRI *`)
9. Test Lambda manually → verify in CloudWatch Logs

---

## 🔗 References

- [Two-Tier Flask App Repo](https://github.com/LondheShubham153/two-tier-flask-app)
- [AWS RDS Documentation](https://docs.aws.amazon.com/rds/)
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [Python Boto3 Docs](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
- [Start/Stop EC2 with Lambda + EventBridge](https://repost.aws/knowledge-center/start-stop-lambda-eventbridge)

---

## 🔗 Connect With Me

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/aishwary-gupta-b734a2327)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black?style=for-the-badge&logo=github)](https://github.com/Aishwary-gupta)

---

## 📅 7 Days of AWS Challenge Progress

| Day | Topic | Status |
|---|---|---|
| Day 1 | AWS Basics | ✅ Done |
| Day 2 | AWS WAF | ✅ Done |
| Day 3 | S3, IAM & CLI | ✅ Done |
| Day 4 | RDS, DynamoDB & Lambda | ✅ Done |
| Day 5 | Coming Soon | 🔄 |
| Day 6 | Coming Soon | 🔄 |
| Day 7 | Coming Soon | 🔄 |

---

> Made with ❤️ as part of **#7DaysOfAWS** challenge by **@TrainWithShubham**

**#7DaysOfAWS #AWSwithTWS #AWS #CloudComputing #RDS #Lambda #DynamoDB #EC2 #DevOps #Serverless**
