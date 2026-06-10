---
description: "Use Supabase PostgreSQL with FastReact - Open-source Firebase alternative with PostgreSQL and a generous free tier."
keywords: "supabase postgres, supabase deployment, fastreact supabase, supabase database"
---

# Supabase PostgreSQL

Use [Supabase](https://supabase.com) as your PostgreSQL database provider for FastReact. Supabase offers a generous free tier and production-ready managed Postgres.

## Why Supabase?

- **Generous free tier** - 500 MB database, 1GB file storage, 50K monthly active users
- **Standard PostgreSQL** - Full Postgres compatibility
- **Open source** - Self-hostable if needed
- **Great dashboard** - Excellent table editor and SQL editor
- **Built-in features** - Auth, Storage, Edge Functions (optional - FastReact uses its own backend)

!!! note "Using Supabase with FastReact"

    FastReact uses Supabase **only** as a PostgreSQL database provider. We don't use Supabase Auth, Supabase realtime, or other Supabase features — FastReact has its own backend for all of that. You're just using Supabase as a managed Postgres host.

## Cost Estimate

- Free tier: **$0/month** (great for development and small projects)
- Pro plan: $25/month (production-ready, 8 GB database)

## Setup

### 1. Create Supabase Account

Sign up at [supabase.com](https://supabase.com) and create a new project.

### 2. Get Connection String

1. In your Supabase dashboard, go to **Settings** → **Database**
2. Under "Connection string", select **URI** format
3. Copy the connection string

### 3. Configure FastReact

Add to `backend/.env`:

```bash
FR_DB_URL=postgresql://postgres:your-password@db.xxx.supabase.co:5432/postgres
FR_DB_SCHEMA=your_schema_name
```

### 4. Run Migrations

```bash
cd backend/db
./sqitch.sh prod deploy
```

## Connection Pooling

For production, use Supabase's transaction pooler (Supavisor):

1. In Supabase dashboard → **Settings** → **Database**
2. Under "Connection pooling", copy the pooler connection string
3. Use this for `FR_DB_URL` in production

## Key Features

- Managed PostgreSQL 15
- Point-in-time recovery (Pro plan)
- Automated daily backups
- Read replicas (Pro plan)
- Connection pooling via Supavisor
- Visual database editor

---

For more: [Supabase Documentation](https://supabase.com/docs)
