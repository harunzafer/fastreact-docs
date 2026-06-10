---
description: "Deploy FastReact on Render - Simple full-stack deployment with containers and managed PostgreSQL."
keywords: "render deployment, render postgres, render docker, fastreact render"
---

# Deploy to Render

Deploy FastReact on [Render](https://render.com) — a simple, developer-friendly platform for containers and managed databases.

## What You'll Use

- **Render Web Service** - Deploy your FastAPI backend container
- **Render PostgreSQL** - Managed PostgreSQL database
- **Render Static Sites** - Host your React frontend

## Cost Estimate

- Web Service (Starter): $7/month
- PostgreSQL (Starter, 1 GB): $7/month
- Static Sites: **Free**

**Total:** ~$14/month (great value!)

## Why Render?

- **Simple setup** - Connect GitHub, it just works
- **Affordable** - One of the cheapest managed options
- **Auto-deploys** - Push to Git, Render deploys automatically
- **Good DX** - Clean dashboard, helpful logs
- **Zero DevOps** - No infrastructure management

## Deployment Guide

**Coming soon!** This guide is under development.

In the meantime, Render has excellent documentation:
- [Render Docs](https://render.com/docs)
- [Deploy a Web Service](https://render.com/docs/web-services)
- [Render PostgreSQL](https://render.com/docs/databases)

## Quick Start

1. Create a new **Web Service** on Render dashboard
2. Connect your GitHub repository
3. Render auto-detects the `Dockerfile` in `backend/`
4. Create a **PostgreSQL** database
5. Link the database to your Web Service (Render auto-injects `DATABASE_URL`)
6. Create a **Static Site** from the `frontend/` directory
7. Set `Build Command`: `npm run build`, `Publish Directory`: `build/client`

## Key Features

- Automatic HTTPS/SSL
- Zero-downtime deploys
- Automatic daily database backups
- Preview environments (team plans)
- Easy environment variable management
- Built-in DDoS protection

---

Want to contribute this guide? [Open a PR](https://github.com/harunzafer/fastreact-docs) with your deployment experience!
