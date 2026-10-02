[![Deploy Website to S3](https://github.com/Anshika032/aws-cicd-project/actions/workflows/deploy.yml/badge.svg)](https://github.com/Anshika032/aws-cicd-project/actions/workflows/deploy.yml)

# 🚀 AWS CI/CD Pipeline — GitHub Actions + Amazon S3

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazon-aws)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-blue?logo=github-actions)
![Amazon S3](https://img.shields.io/badge/Amazon%20S3-Storage-green?logo=amazon-s3)
![IAM](https://img.shields.io/badge/AWS%20IAM-Security-red?logo=amazon-aws)
![OIDC](https://img.shields.io/badge/GitHub-OIDC-purple?logo=github)

A simple cloud deployment project that demonstrates how to automatically deploy a static website to **Amazon S3** using **GitHub Actions** and **AWS IAM OIDC authentication**.

The project follows a secure CI/CD workflow where GitHub Actions obtains temporary AWS credentials through OIDC instead of storing long-term AWS access keys in GitHub.

---

## 📌 Project Overview

This project demonstrates a basic **Continuous Integration and Continuous Deployment (CI/CD)** pipeline using AWS.

Whenever changes are pushed to the `main` branch:

1. GitHub detects the new commit.
2. GitHub Actions starts the deployment workflow.
3. GitHub authenticates with AWS using OIDC.
4. AWS STS provides temporary credentials to GitHub Actions.
5. The IAM role grants access only to the required S3 bucket.
6. The website files are synchronized to Amazon S3.
7. The updated website is available in the S3 bucket.

---

## 🏗️ Architecture

```text
                    ┌──────────────────┐
                    │     Developer    │
                    │                  │
                    │ HTML / CSS Code  │
                    └────────┬─────────┘
                             │
                             │ git push
                             ▼
                    ┌──────────────────┐
                    │     GitHub       │
                    │   Repository     │
                    └────────┬─────────┘
                             │
                             │ Push to main
                             ▼
                 ┌────────────────────────┐
                 │    GitHub Actions      │
                 │                        │
                 │  Checkout Repository   │
                 │          ↓             │
                 │  OIDC Authentication   │
                 │          ↓             │
                 │  AWS S3 Sync           │
                 └───────────┬────────────┘
                             │
                             │ Temporary AWS Credentials
                             ▼
                    ┌──────────────────┐
                    │    AWS IAM       │
                    │                  │
                    │ GitHubActions    │
                    │ S3DeployRole     │
                    └────────┬─────────┘
                             │
                             │ Authorized S3 Actions
                             ▼
                    ┌──────────────────┐
                    │   Amazon S3      │
                    │                  │
                    │ index.html       │
                    │ style.css        │
                    └──────────────────┘

Planned Production Architecture
CloudFront is planned as the next layer:
GitHub
   │
   ▼
GitHub Actions
   │
   ▼
AWS IAM OIDC
   │
   ▼
Private Amazon S3
   │
   ▼
CloudFront + OAC
   │
   ▼
HTTPS Website

🛠️ Technologies Used
Technology	Purpose
GitHub	Source code repository
GitHub Actions	CI/CD automation
AWS IAM	Identity and access management
GitHub OIDC	Keyless AWS authentication
AWS STS	Temporary AWS credentials
Amazon S3	Website file storage
HTML5	Website structure
CSS3	Website styling


📂 Project Structure
aws-cicd-project/
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
├── index.html
├── style.css
└── README.md

⚙️ GitHub Actions Workflow
The deployment workflow is stored at:
.github/workflows/deploy.yml

name: Deploy Website to S3

on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::146259919986:role/GitHubActionsS3DeployRole
          aws-region: ap-south-1

      - name: Deploy website to S3
        run: |
          aws s3 sync . s3://anshika-aws-cicd-2026 --delete \
            --exclude ".git/*" \
            --exclude ".github/*"

🔐 Authentication with GitHub OIDC
This project does not store AWS access keys inside GitHub.
Instead, GitHub Actions uses OpenID Connect (OIDC) to authenticate with AWS.
The authentication flow is:
GitHub Actions
      │
      │ OIDC Token
      ▼
GitHub OIDC Provider
      │
      ▼
AWS IAM
      │
      │ AssumeRoleWithWebIdentity
      ▼
GitHubActionsS3DeployRole
      │
      ▼
Temporary AWS Credentials

Why OIDC?
Traditional CI/CD systems may use long-term AWS access keys.
This project uses OIDC so that GitHub Actions can obtain temporary credentials without storing permanent AWS secret keys in GitHub.
🔑 IAM Security
The GitHub Actions role is:
GitHubActionsS3DeployRole

The role uses a custom inline policy restricted to the project bucket.
Bucket-level permission
{
  "Effect": "Allow",
  "Action": [
    "s3:ListBucket"
  ],
  "Resource": "arn:aws:s3:::anshika-aws-cicd-2026"
}

Object-level permissions
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:PutObject",
    "s3:DeleteObject"
  ],
  "Resource": "arn:aws:s3:::anshika-aws-cicd-2026/*"
}

The role is therefore limited to the S3 resources required by the deployment workflow.
🪣 Amazon S3 Configuration
The website is stored in:
anshika-aws-cicd-2026

AWS Region:
Asia Pacific (Mumbai)
ap-south-1

The bucket is configured with:
- ✅ Versioning enabled
- ✅ Bucket owner enforced
- ✅ ACLs disabled
- ✅ Block Public Access enabled
- ✅ Server-side encryption using SSE-S3
- ✅ Bucket Key enabled
- ✅ Private bucket configuration
The current website files are:
index.html
style.css

🔄 Deployment Process
A deployment happens automatically whenever code is pushed to main.
1. Developer modifies website
              ↓
2. Commit pushed to GitHub
              ↓
3. GitHub Actions workflow starts
              ↓
4. Repository is checked out
              ↓
5. GitHub OIDC authenticates with AWS
              ↓
6. IAM role is assumed
              ↓
7. Temporary AWS credentials are issued
              ↓
8. AWS S3 Sync runs
              ↓
9. Website files are updated in S3

🧪 CI/CD Test
The pipeline was tested by modifying the website and pushing the change to GitHub.
GitHub Actions successfully completed the deployment workflow.
Deployment result
Deploy Website to S3
        ✅ Success

The updated files were then visible inside the S3 bucket.
This confirms that the automated deployment pipeline is functioning.
🔄 S3 Synchronization
The deployment uses:
aws s3 sync . s3://anshika-aws-cicd-2026 --delete

with:
--exclude ".git/*"
--exclude ".github/*"

The sync command compares the local repository with the S3 bucket and uploads changed files.
The --delete option removes files from S3 that no longer exist in the deployment source.
☁️ CloudFront Integration
The planned final architecture uses Amazon CloudFront in front of the private S3 bucket.
CloudFront will provide:
- HTTPS access
- CDN delivery
- Global edge caching
- Private S3 origin access through Origin Access Control (OAC)
Planned flow
User
 │
 │ HTTPS
 ▼
CloudFront
 │
 │ OAC
 ▼
Private S3 Bucket
 │
 ├── index.html
 └── style.css

CloudFront creation is currently pending AWS account-level support because the AWS console is displaying an account verification restriction even though Customer Verification is shown as verified.
An AWS Support case has been submitted for this issue.
📊 Current Project Status
Component	Status
GitHub Repository	✅ Complete
HTML Website	✅ Complete
CSS Styling	✅ Complete
GitHub Actions	✅ Complete
GitHub OIDC	✅ Complete
AWS IAM Role	✅ Complete
Least-Privilege S3 Policy	✅ Complete
Amazon S3 Deployment	✅ Complete
Automatic Deployment Test	✅ Passed
CloudFront	⏳ Pending AWS Support
HTTPS Website	⏳ Pending CloudFront


🎯 Learning Objectives
This project demonstrates practical knowledge of:
- Cloud computing fundamentals
- AWS S3
- AWS IAM
- IAM roles
- IAM policies
- GitHub Actions
- CI/CD pipelines
- OpenID Connect
- Temporary AWS credentials
- Infrastructure security
- Automated deployments
- Static website hosting architecture
- AWS CloudFront
- Origin Access Control
🚀 Future Improvements
Planned improvements include:
- [ ] Complete CloudFront integration
- [ ] Configure Origin Access Control
- [ ] Configure HTTPS delivery
- [ ] Add CloudFront caching
- [ ] Add custom domain using Route 53
- [ ] Add deployment status badge
- [ ] Add automated tests before deployment
- [ ] Add deployment notifications
- [ ] Add CloudWatch monitoring
👩‍💻 Author
Anshika Shukla
Electronics & Communication Engineering Student
GitHub: Anshika032
📜 License
This project is created for educational and portfolio purposes.
