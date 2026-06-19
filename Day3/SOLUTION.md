# 🗂️ Day 3 — S3, IAM & AWS CLI

> **7 Days of AWS Challenge** | Day 3 of 7

![AWS](https://img.shields.io/badge/AWS-S3-orange?style=for-the-badge&logo=amazon-aws)
![IAM](https://img.shields.io/badge/AWS-IAM-yellow?style=for-the-badge&logo=amazon-aws)
![CLI](https://img.shields.io/badge/AWS-CLI-green?style=for-the-badge&logo=amazon-aws)
![EC2](https://img.shields.io/badge/AWS-EC2-blue?style=for-the-badge&logo=amazon-aws)
![Status](https://img.shields.io/badge/Status-Completed-green?style=for-the-badge)

---

## 📌 Project Overview

In this project, I learned how to securely store data on AWS S3, manage user access using IAM policies, and perform cloud operations entirely from the **AWS CLI** on Windows — without touching the AWS Console.

---

## 🏗️ Architecture

IAM User (Aishwarygupta-admin)

│

▼

AWS CLI (Windows)

│

├──▶ S3 Private Bucket (globaltech-day3-aishwary-2026)

│         └── Bucket Policy (Allow only IAM user)

│

└──▶ EC2 Instance (Day3-EC2, t3.micro)

└── Launched & Stopped via CLI
IAM User (Alex)

└── Custom Policy (AlexCustomPolicy)

├── EC2 Read Only ✅

├── S3 Create Bucket ✅

└── EC2 Modify/Terminate ❌

---

## ✅ What I Completed

- [x] Created IAM user for secure daily access (no root usage)
- [x] Created private S3 bucket with Block Public Access enabled
- [x] Attached custom bucket policy — only IAM user can access
- [x] Successfully uploaded a file to the private bucket
- [x] Verified file is private (Access Denied on direct URL)
- [x] Installed and configured AWS CLI v2 on Windows
- [x] Verified CLI with `aws sts get-caller-identity`
- [x] Fetched Amazon Linux 2 AMI ID using CLI
- [x] Created key pair `day3-keypair` via CLI
- [x] Launched EC2 `t3.micro` instance named `Day3-EC2` via CLI
- [x] Verified instance state using `describe-instances`
- [x] Stopped EC2 instance via CLI to avoid charges
- [x] Created IAM user `Alex` with custom least-privilege policy

---

## 🔧 Tech Stack

| Service | Purpose |
|---|---|
| AWS S3 | Private object storage |
| AWS IAM | User and access management |
| AWS CLI v2 | Terminal-based cloud management |
| AWS EC2 (t3.micro) | Cloud compute instance |
| Windows CMD | CLI environment |

---

## 🛡️ Bucket Policy Used

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowMyUser",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::866435873023:user/Aishwarygupta-admin"
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::globaltech-day3-aishwary-2026",
        "arn:aws:s3:::globaltech-day3-aishwary-2026/*"
      ]
    }
  ]
}
```

---

## 👥 IAM Policy for Alex (AlexCustomPolicy)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EC2ReadOnly",
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "ec2:List*"
      ],
      "Resource": "*"
    },
    {
      "Sid": "S3CreateBuckets",
      "Effect": "Allow",
      "Action": [
        "s3:CreateBucket",
        "s3:ListAllMyBuckets",
        "s3:GetBucketLocation"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 🖥️ CLI Commands Used

### Setup & Verify
```bash
aws --version
aws configure
aws sts get-caller-identity
```

### S3
```bash
aws s3 ls
aws s3 ls s3://globaltech-day3-aishwary-2026
```

### EC2
```bash
# Get AMI ID
aws ec2 describe-images --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" \
  "Name=state,Values=available" \
  --query "sort_by(Images,&CreationDate)[-1].ImageId" \
  --output text

# Create Key Pair
aws ec2 create-key-pair --key-name day3-keypair \
  --query "KeyMaterial" --output text > day3-keypair.pem

# Launch Instance
aws ec2 run-instances --image-id ami-05726d1f069460e90 \
  --instance-type t3.micro --key-name day3-keypair --count 1 \
  --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=Day3-EC2}]"

# Verify State
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=Day3-EC2" \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]" \
  --output table

# Stop Instance
aws ec2 stop-instances --instance-ids i-XXXXXXXXXXXXXXXXX
```

### IAM
```bash
aws iam create-user --user-name Alex
aws iam create-policy --policy-name AlexCustomPolicy \
  --policy-document file://alex-policy.json
aws iam attach-user-policy --user-name Alex \
  --policy-arn arn:aws:iam::866435873023:policy/AlexCustomPolicy
aws iam list-attached-user-policies --user-name Alex
```

---

## 📸 Screenshots

### S3 Bucket Created
![S3 Bucket](screenshots/s3-bucket.png)

### Block Public Access ON
![Block Public Access](screenshots/block-public-access.png)

### Bucket Policy Saved
![Bucket Policy](screenshots/bucket-policy.png)

### File Uploaded Successfully
![File Uploaded](screenshots/file-uploaded.png)

### Access Denied on Direct URL
![Access Denied](screenshots/access-denied.png)

### CLI Configured Successfully
![CLI Config](screenshots/cli-configured.png)

### EC2 Instance Running via CLI
![EC2 Running](screenshots/ec2-running.png)

### IAM User Alex with Policy
![Alex Policy](screenshots/alex-policy.png)

---

## 💡 Key Learnings

- **Never use root account** for daily tasks — always create an IAM user
- **S3 bucket policies** control exactly who can read/write your data
- **Block Public Access** should always be ON for private buckets
- **Presigned URLs** allow temporary access to private files without making them public
- **AWS CLI** is more powerful than the console — everything can be automated
- **Principle of Least Privilege** — give users only what they need, nothing more
- **t3.micro** is free tier eligible in Mumbai (ap-south-1) — not t2.micro
- Always **stop EC2 instances** after testing to avoid charges
- **IAM is completely free** — no charges for users, policies or groups

---

## 🔐 Security Best Practices Followed

- Created IAM user instead of using root account
- Enabled Block Public Access on S3 bucket
- Used bucket policy to restrict access to specific IAM user only
- Applied Principle of Least Privilege for Alex's permissions
- Stopped EC2 instance immediately after testing
- Downloaded and secured key pair (.pem file)

---

## 💰 Cost Breakdown

| Resource | Cost |
|---|---|
| S3 Bucket (small file) | Free tier |
| IAM Users & Policies | Always free |
| EC2 t3.micro (stopped) | Free tier |
| AWS CLI | Free |
| **Total** | **$0** |

---


---

## 🚀 How to Reproduce

1. Create IAM user with AdministratorAccess
2. Create private S3 bucket with Block Public Access ON
3. Attach bucket policy allowing only your IAM user
4. Install AWS CLI v2 on your machine
5. Run `aws configure` with your access keys
6. Launch EC2 using `aws ec2 run-instances`
7. Create IAM user Alex with custom least-privilege policy
8. Stop EC2 after testing

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
| Day 4 | Coming Soon | 🔄 |
| Day 5 | Coming Soon | 🔄 |
| Day 6 | Coming Soon | 🔄 |
| Day 7 | Coming Soon | 🔄 |

---

> Made with ❤️ as part of **#7DaysOfAWS** challenge by **@TrainWithShubham**

**#7DaysOfAWS #AWSwithTWS #AWS #CloudComputing #S3 #IAM #AWSCLI #EC2 #DevOps**
