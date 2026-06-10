---
description: "Deploy FastReact on Google Cloud - Cloud Run for backend, Cloud SQL for PostgreSQL, Cloud Storage for frontend hosting."
keywords: "google cloud deployment, cloud run, cloud sql postgresql, gcp deployment, fastreact gcp"
---

# Deploy to Google Cloud

Deploy FastReact on Google Cloud Platform using serverless container and managed database services.

## What You'll Use

- **Cloud Run** - Serverless containers for FastAPI backend
- **Cloud SQL (PostgreSQL)** - Managed relational database
- **Cloud Storage + Firebase Hosting** - Static site hosting for React frontend
- **Artifact Registry** - Store Docker images

## Cost Estimate

For a small production app:
- Cloud Run (request-based pricing): ~$0-10/month
- Cloud SQL (db-f1-micro): ~$10-20/month
- Cloud Storage + Firebase Hosting: ~$1-5/month

**Total:** ~$11-35/month (Cloud Run scales to zero when idle)

## Why Google Cloud?

- **Serverless scaling** - Cloud Run scales to zero and up automatically
- **Cost-effective** - Pay only for actual usage
- **Fast deployments** - Container builds and deploys in minutes
- **Integrated services** - Cloud SQL, Secret Manager, Cloud Run work seamlessly
- **Great free tier** - Generous free usage limits

## Deployment Guide

**Coming soon!** This guide is under development.

In the meantime, GCP has excellent documentation:
- [Cloud Run Quickstart](https://cloud.google.com/run/docs/quickstarts)
- [Cloud SQL for PostgreSQL](https://cloud.google.com/sql/docs/postgres)
- [Firebase Hosting](https://firebase.google.com/docs/hosting)

## Quick Start

```bash
# Install gcloud CLI
# https://cloud.google.com/sdk/docs/install

# Login
gcloud auth login
gcloud config set project YOUR_PROJECT_ID

# Build and deploy backend
cd backend
gcloud run deploy fastreact-api \
  --source . \
  --region us-central1 \
  --allow-unauthenticated

# Deploy frontend to Firebase Hosting
cd frontend
npm run build
firebase deploy --only hosting
```

## Key Features

- Scale to zero when no traffic
- Automatic HTTPS/SSL
- Global load balancing
- Cloud SQL Auth Proxy for secure DB connections
- Secret Manager integration
- Cloud Monitoring and Logging

---

Want to contribute this guide? [Open a PR](https://github.com/harunzafer/fastreact-docs) with your deployment experience!
