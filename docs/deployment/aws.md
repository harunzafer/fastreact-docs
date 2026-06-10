---
description: "Deploy FastReact on AWS - ECS/Fargate for containers, RDS for PostgreSQL, S3/CloudFront for frontend hosting."
keywords: "aws deployment, ecs fargate, aws rds postgresql, aws cloudfront, fastreact aws"
---

# Deploy to AWS

Deploy FastReact on Amazon Web Services using managed container, database, and CDN services.

## What You'll Use

- **ECS Fargate** - Serverless container deployment for FastAPI backend
- **RDS PostgreSQL** - Managed relational database
- **S3 + CloudFront** - Static site hosting for React frontend
- **ECR** - Elastic Container Registry for Docker images

## Cost Estimate

For a small production app:
- ECS Fargate (0.25 vCPU, 0.5GB): ~$10-15/month
- RDS PostgreSQL (db.t3.micro): ~$15-25/month
- S3 + CloudFront: ~$1-5/month
- ECR: ~$1/month

**Total:** ~$27-46/month

## Why AWS?

- **Industry standard** - Most widely used cloud platform
- **Comprehensive services** - Everything you need in one ecosystem
- **High reliability** - 99.99% SLA on most services
- **Global scale** - Regions worldwide
- **Enterprise-ready** - Compliance, security, and governance built-in

## Deployment Guide

**Coming soon!** This guide is under development.

In the meantime, AWS has excellent documentation:
- [ECS Fargate Getting Started](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/getting-started-fargate.html)
- [RDS PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreatePostgreSQLInstance.html)
- [Host Static Website on S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)

## Architecture Overview

```
Route 53 (DNS)
  ↓
CloudFront (CDN)
  ↓
S3 (Frontend static files) + ALB → ECS Fargate (Backend) → RDS PostgreSQL
```

## Key Features

- Auto-scaling containers
- Multi-AZ database availability
- Global CDN with CloudFront
- IAM role-based security
- VPC network isolation
- AWS Secrets Manager integration

---

Want to contribute this guide? [Open a PR](https://github.com/harunzafer/fastreact-docs) with your deployment experience!
