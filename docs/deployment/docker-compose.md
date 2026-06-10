---
description: "Deploy FastReact with Docker Compose - Self-hosted deployment on your own server or VPS."
keywords: "docker compose deployment, self-hosted fastreact, vps deployment"
---

# Deploy with Docker Compose

Deploy FastReact on your own server using Docker Compose for full control over your infrastructure.

## What You'll Need

- A Linux server (Ubuntu 22.04+ recommended)
- Docker and Docker Compose installed
- A domain name (optional but recommended)
- Basic Linux and DevOps knowledge

## Cost Estimate

- VPS (2GB RAM): $5-12/month
- Domain: $10-15/year
- **Total:** ~$5-15/month

## Why Docker Compose?

- **Full control** - Your server, your rules
- **Cost-effective** - Cheapest option for single deployment
- **Simple** - One configuration file
- **Portable** - Works on any Docker host
- **No vendor lock-in** - Standard Docker setup

## Prerequisites

```bash
# Install Docker
curl -fsSL https://get.docker.com | sh

# Install Docker Compose
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

## FastReact Includes docker-compose.yml

FastReact repository includes a `docker-compose.yml` file as a starting point for deployment.

```yaml
# Example structure (simplified)
services:
  db:
    image: postgres:16
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: fastreact
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${DB_PASSWORD}

  api:
    build: ./backend
    depends_on:
      - db
    environment:
      FR_DB_URL: postgresql://postgres:${DB_PASSWORD}@db:5432/fastreact
      FR_ENVIRONMENT: prod
    ports:
      - "8000:8000"

  frontend:
    build: ./frontend
    ports:
      - "80:80"
      - "443:443"

volumes:
  postgres_data:
```

## Production Setup

### 1. Clone Your Repository

```bash
git clone https://github.com/your-org/your-project.git
cd your-project
```

### 2. Configure Environment

Create `backend/.env` with production values:

```bash
FR_ENVIRONMENT=prod
FR_DB_URL=postgresql://postgres:yourpassword@db:5432/fastreact
FR_BASE_WEB_URL=https://app.yourdomain.com
FR_BASE_API_URL=https://api.yourdomain.com
FR_EMAIL_PROVIDER=resend
FR_RESEND_API_KEY=re_your_key
FR_STRIPE_API_KEY=sk_live_your_key
FR_STRIPE_WEBHOOK_SECRET=whsec_your_secret
```

### 3. Start Services

```bash
# Start all services
docker compose up -d

# Run database migrations
docker compose exec api sh -c "cd /app/db && ./sqitch.sh prod deploy"

# Create admin user
docker compose exec api sh -c "uv run scripts/create_admin.py"
```

### 4. Set Up HTTPS

Use [Caddy](https://caddyserver.com) or [Nginx + Certbot](https://certbot.eff.org) as a reverse proxy for HTTPS:

**Caddyfile (recommended):**

```
api.yourdomain.com {
    reverse_proxy api:8000
}

app.yourdomain.com {
    root * /srv/frontend
    file_server
}
```

## Updating Your Deployment

```bash
# Pull latest code
git pull origin main

# Rebuild and restart
docker compose build
docker compose up -d

# Run any new migrations
docker compose exec api sh -c "cd /app/db && ./sqitch.sh prod deploy"
```

## Key Features

- Full control over infrastructure
- Easy rollbacks with `docker compose`
- Customizable reverse proxy configuration
- Supports multiple services on one server

---

Want to contribute this guide? [Open a PR](https://github.com/harunzafer/fastreact-docs) with your deployment experience!
