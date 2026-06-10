---
description: "Deploy FastReact frontend and landing page on Vercel - Fast, simple deployment for React Router v7 applications."
keywords: "vercel deployment, vercel react, fastreact vercel, vercel frontend"
---

# Deploy to Vercel

Deploy your FastReact frontend and landing page on Vercel for blazing-fast static site hosting with excellent developer experience.

## What You'll Use

- **Vercel** - Frontend and landing page hosting
- **Backend elsewhere** - Use Azure, Railway, Fly.io, or DigitalOcean for backend
- **Database elsewhere** - Use Neon, Supabase, or another provider

**Note:** Vercel is perfect for the frontend, but you'll need separate hosting for your FastAPI backend container.

## Cost Estimate

- Vercel Hobby (personal): **Free**
- Vercel Pro (production): ~$20/month
- Plus backend hosting: $10-30/month
- Plus database: $10-20/month

**Total:** ~$40-70/month (Free + $20-50 for backend/DB)

## Why Vercel?

- **Best DX** - Deploy with `git push`
- **Fast** - Global edge network
- **Preview deployments** - Every PR gets a URL
- **Zero config** - Works with React Router v7 out of the box
- **Great free tier** - Perfect for side projects

## Deployment Guide

### 1. Build the Frontend

```bash
cd frontend
npm run build
```

The output goes to `build/client/` for static assets.

### 2. Deploy Frontend

```bash
# Install Vercel CLI
npm i -g vercel

# Login
vercel login

# Deploy from the frontend directory
vercel --prod
```

Or connect your GitHub repository at [vercel.com/new](https://vercel.com/new) for automatic deployments on every push.

### 3. Configure Environment Variables

In the Vercel dashboard, add your frontend environment variables:

```bash
VITE_API_URL=https://api.yourdomain.com
VITE_APP_MODE=b2c  # or b2b
```

### 4. Configure Build Settings

In your Vercel project settings:
- **Build Command**: `npm run build`
- **Output Directory**: `build/client`
- **Install Command**: `npm install`

## Custom Domain

1. Go to your project in Vercel dashboard
2. Click **Settings** → **Domains**
3. Add your domain (e.g., `app.yourdomain.com`)
4. Update DNS records as instructed

## Deploy Landing Page

The landing page is a separate app. Deploy it the same way from the `landing/` directory to a separate Vercel project (e.g., `fastreact.dev`).

## Key Features

- Automatic HTTPS/SSL
- Preview deployments for every PR
- Instant cache invalidation
- Built-in analytics
- Edge functions support (if needed)

---

Want to contribute improvements? [Open a PR](https://github.com/harunzafer/fastreact-docs) with your deployment experience!
