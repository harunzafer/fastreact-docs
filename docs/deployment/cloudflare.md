---
description: "Deploy FastReact frontend on Cloudflare Pages - Global edge deployment for React Router v7 applications."
keywords: "cloudflare pages, cloudflare deployment, fastreact cloudflare, edge deployment"
---

# Deploy to Cloudflare Pages

Deploy your FastReact frontend on Cloudflare Pages for ultra-fast global edge hosting.

## What You'll Use

- **Cloudflare Pages** - Frontend hosting on Cloudflare's edge network
- **Backend elsewhere** - FastAPI backend on Azure, Railway, or similar

## Cost Estimate

- Cloudflare Pages: **Free** (unlimited requests, 500 builds/month)
- Plus backend hosting: $10-30/month

**Total:** ~$10-30/month (one of the cheapest options!)

## Why Cloudflare Pages?

- **Blazing fast** - Runs at 275+ edge locations worldwide
- **Generous free tier** - Unlimited requests
- **Zero config** - Works with React out of the box
- **DDoS protection** - Built-in security
- **Workers integration** - Edge functions if needed

## Deployment Guide

### 1. Build the Frontend

```bash
cd frontend
npm run build
```

### 2. Deploy via Wrangler CLI

```bash
# Install Wrangler
npm install -g wrangler

# Login
wrangler login

# Deploy
cd frontend
wrangler pages deploy build/client --project-name fastreact
```

### 3. Or Deploy via Dashboard

1. Go to [dash.cloudflare.com](https://dash.cloudflare.com) → **Pages**
2. Click **Create a project** → **Connect to Git**
3. Select your repository
4. Configure:
   - **Build command**: `npm run build`
   - **Build output directory**: `build/client`
   - **Root directory**: `frontend`
5. Click **Save and Deploy**

### 4. Configure Environment Variables

In Pages project settings → **Environment variables**:

```bash
VITE_API_URL=https://api.yourdomain.com
VITE_APP_MODE=b2c
```

## Custom Domain

1. Pages project → **Custom domains**
2. Add your domain
3. Cloudflare handles DNS automatically if your domain uses Cloudflare nameservers

## Key Features

- Global edge network (275+ locations)
- Unlimited free requests
- Preview deployments for every PR
- Automatic HTTPS
- DDoS protection included
- Web Analytics (free)

---

Want to contribute this guide? [Open a PR](https://github.com/harunzafer/fastreact-docs) with your deployment experience!
