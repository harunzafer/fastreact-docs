---
description: "Deploy FastReact frontend on Netlify - Simple static site hosting with great developer experience."
keywords: "netlify deployment, netlify react, fastreact netlify, netlify frontend"
---

# Deploy to Netlify

Deploy your FastReact frontend on Netlify for fast, reliable static hosting with a great developer experience.

## What You'll Use

- **Netlify** - Frontend and landing page hosting
- **Backend elsewhere** - Use Azure, Railway, Fly.io, or DigitalOcean for backend

## Cost Estimate

- Netlify Free: **Free** (100GB bandwidth/month)
- Netlify Pro: ~$19/month
- Plus backend hosting: $10-30/month

**Total:** ~$30-50/month

## Why Netlify?

- **Easy setup** - Connect GitHub, done
- **Fast global CDN** - Edge hosting worldwide
- **Instant rollbacks** - One-click revert
- **Branch previews** - Every PR gets a preview URL
- **Forms and functions** - Built-in serverless capabilities

## Deployment Guide

### 1. Build the Frontend

```bash
cd frontend
npm run build
```

Output goes to `build/client/`.

### 2. Deploy with Netlify CLI

```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login
netlify login

# Deploy
cd frontend
netlify deploy --prod --dir build/client
```

### 3. Or Deploy via GitHub

1. Go to [app.netlify.com](https://app.netlify.com) → **Add new site**
2. Connect your GitHub repository
3. Configure:
   - **Base directory**: `frontend`
   - **Build command**: `npm run build`
   - **Publish directory**: `build/client`
4. Click **Deploy site**

### 4. Configure Environment Variables

In Netlify dashboard → **Site settings** → **Environment variables**:

```bash
VITE_API_URL=https://api.yourdomain.com
VITE_APP_MODE=b2c
```

## Custom Domain

1. Netlify dashboard → **Domain management**
2. Add your custom domain
3. Update DNS to point to Netlify

## Key Features

- Automatic HTTPS/SSL
- Deploy previews for pull requests
- Form submissions (no server needed)
- Split testing
- Analytics

---

Want to contribute this guide? [Open a PR](https://github.com/harunzafer/fastreact-docs) with your deployment experience!
