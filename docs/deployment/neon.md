---
description: "Use Neon serverless PostgreSQL with FastReact - Affordable, scalable database with a generous free tier."
keywords: "neon postgres, neon deployment, serverless postgresql, fastreact database, neon free tier"
---

# Neon PostgreSQL

Use [Neon](https://neon.tech) as your managed PostgreSQL database for FastReact. Neon is a serverless Postgres provider with a generous free tier and affordable scaling.

## Why Neon?

- **Serverless** - Scales to zero when idle, no wasted compute
- **Generous free tier** - 0.5 GB storage, 1 compute unit free
- **Branching** - Create database branches for dev/staging (like git branches!)
- **Affordable** - $19/month for production use
- **Standard Postgres** - Works with any Postgres client

## Cost Estimate

- Free tier: **$0/month** (great for development)
- Launch plan: $19/month (production-ready)

## Setup

### 1. Create Neon Account

Sign up at [neon.tech](https://neon.tech) and create a new project.

### 2. Get Connection String

1. In your Neon dashboard, select your project
2. Click **Connect** or **Connection Details**
3. Copy the connection string (starts with `postgresql://`)

### 3. Configure FastReact

Add to `backend/.env`:

```bash
FR_DB_URL=postgresql://username:password@ep-xxx.us-east-2.aws.neon.tech/neondb?sslmode=require
FR_DB_SCHEMA=your_schema_name
```

### 4. Run Migrations

```bash
cd backend/db
./sqitch.sh prod deploy
```

## Neon Database Branches

Neon's branching feature is perfect for FastReact:

```bash
# Create a branch for staging
neon branches create --name staging

# Use different connection strings per environment
# dev: main branch
# staging: staging branch
# prod: production branch
```

## Connection Pooling

For production, use Neon's built-in connection pooler (PgBouncer) by adding `-pooler` to your endpoint:

```bash
FR_DB_URL=postgresql://username:password@ep-xxx-pooler.us-east-2.aws.neon.tech/neondb?sslmode=require
```

## Key Features

- Autoscaling compute
- Point-in-time recovery (14 days)
- Database branching
- Read replicas (paid plans)
- Built-in connection pooling
- Standard PostgreSQL 16

---

For more: [Neon Documentation](https://neon.tech/docs)
