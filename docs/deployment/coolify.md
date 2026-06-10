---
description: "Deploy FastReact with Coolify - Self-hosted PaaS for full control over your infrastructure."
keywords: "coolify deployment, self-hosted paas, fastreact coolify, open source heroku"
---

# Deploy with Coolify

Deploy FastReact using [Coolify](https://coolify.io) — an open-source, self-hostable PaaS that works like your own Heroku.

## What is Coolify?

Coolify is an open-source platform you install on your own VPS. It gives you a beautiful UI to deploy apps, manage databases, and configure domains — without writing infrastructure code.

## What You'll Need

- A VPS with at least 2 GB RAM (Ubuntu 22.04+ recommended)
- Coolify installed ([installation guide](https://coolify.io/docs/installation))
- Your FastReact repository on GitHub/GitLab

## Cost Estimate

- VPS (2-4 GB RAM): $12-24/month (Hetzner, DigitalOcean, etc.)
- Coolify: **Free and open source**
- Domain: ~$10-15/year

**Total:** ~$12-24/month — full control, no per-service pricing

## Why Coolify?

- **Full control** - Your server, your data
- **One server, many apps** - Run FastReact and other projects on one VPS
- **Beautiful UI** - No command-line required for deployments
- **Open source** - No vendor lock-in, self-hostable
- **Cost-effective** - Pay for VPS, not per-app

## Deployment Guide

**Coming soon!** This guide is under development.

In the meantime, Coolify has excellent documentation:
- [Coolify Documentation](https://coolify.io/docs)
- [Deploy Applications](https://coolify.io/docs/applications/overview)
- [Deploy Databases](https://coolify.io/docs/databases/overview)

## Quick Start

1. Install Coolify on your VPS: `curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash`
2. Access the Coolify dashboard at `http://your-server-ip:8000`
3. Create a new **Application** and connect your GitHub repository
4. Coolify auto-detects `Dockerfile` in `backend/`
5. Create a **PostgreSQL** database resource in Coolify
6. Link database to your application via environment variables
7. Create another Application for the React frontend (static build)
8. Set up your domain and enable SSL in Coolify

## Key Features

- One-click SSL certificates (Let's Encrypt)
- Git-based deployments
- Database management UI
- Real-time logs
- Automatic backups
- Multiple team members support

---

Want to contribute this guide? [Open a PR](https://github.com/harunzafer/fastreact-docs) with your deployment experience!
