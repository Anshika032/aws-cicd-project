# AWS CI/CD Pipeline — GitHub Actions + Amazon S3 + CloudFront

A simple cloud-based CI/CD pipeline that automatically deploys a static website to Amazon S3 whenever changes are pushed to the GitHub `main` branch.

The project demonstrates GitHub Actions, AWS IAM, OpenID Connect (OIDC), Amazon S3, and CloudFront.

---

## 🚀 Project Overview

This project implements an automated deployment pipeline:

```text
Developer
    │
    │ git push
    ▼
GitHub Repository
    │
    ▼
GitHub Actions
    │
    │ OIDC authentication
    ▼
AWS IAM Role
    │
    │ temporary AWS credentials
    ▼
Amazon S3
    │
    │
    ▼
CloudFront
    │
    ▼
HTTPS Website
