---
description: "Deploy FastReact on Azure - Container Apps for backend, PostgreSQL Flexible Server for database, and Static Web Apps for frontend."
keywords: "azure deployment, azure container apps, azure postgresql, azure static web apps, fastreact azure"
---

# Deploy to Azure

Deploy FastReact using Azure's modern container and serverless services. This guide covers the essential Azure resources you'll need.

## What You'll Use

- **Azure Container Apps** - Run your FastAPI backend container
- **PostgreSQL Flexible Server** - Managed PostgreSQL database
- **Azure Static Web Apps** - Host your React frontend and landing page
- **Azure Container Registry** - Store your Docker images (built-in)

## Prerequisites

- Azure account with active subscription
- Azure CLI installed ([install guide](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli))
- Docker installed locally
- Your FastReact codebase ready

## Cost Estimate

For a small production app:
- Container Apps: ~$15-30/month
- PostgreSQL Flexible Server: ~$50-100/month (can use [Neon](neon.md) for ~$20/month instead)
- Static Web Apps: Free tier available, ~$10/month for standard
- Container Registry: ~$5/month

**Total:** ~$70-145/month (or ~$50/month with Neon for database)

## Architecture

```
Internet
  ↓
Azure Front Door (optional CDN)
  ↓
Static Web Apps (Frontend) → Container Apps (Backend) → PostgreSQL
```

## Deployment Steps

### 1. Login to Azure

```bash
az login
az account set --subscription "Your Subscription Name"
```

### 2. Create Resource Group

```bash
az group create \
  --name fastreact-prod-rg \
  --location eastus
```

### 3. Create Container Registry

```bash
az acr create \
  --resource-group fastreact-prod-rg \
  --name fastreactregistry \
  --sku Basic \
  --admin-enabled true
```

### 4. Build and Push Docker Image

```bash
cd backend

# Login to registry
az acr login --name fastreactregistry

# Build and push
az acr build \
  --registry fastreactregistry \
  --image fastreact-api:latest .
```

### 5. Create PostgreSQL Database

```bash
az postgres flexible-server create \
  --resource-group fastreact-prod-rg \
  --name fastreact-db \
  --location eastus \
  --admin-user fastreactadmin \
  --admin-password "YourSecurePassword123!" \
  --sku-name Standard_B1ms \
  --tier Burstable \
  --version 16
```

### 6. Create Container App Environment

```bash
az containerapp env create \
  --name fastreact-env \
  --resource-group fastreact-prod-rg \
  --location eastus
```

### 7. Deploy Backend Container App

```bash
az containerapp create \
  --name fastreact-api \
  --resource-group fastreact-prod-rg \
  --environment fastreact-env \
  --image fastreactregistry.azurecr.io/fastreact-api:latest \
  --target-port 8000 \
  --ingress external \
  --registry-server fastreactregistry.azurecr.io \
  --env-vars \
    FR_ENVIRONMENT=prod \
    FR_DB_URL="postgresql://fastreactadmin:YourSecurePassword123!@fastreact-db.postgres.database.azure.com/fastreact" \
    FR_BASE_WEB_URL="https://your-frontend.azurestaticapps.net" \
    FR_BASE_API_URL="https://fastreact-api.yourdomain.azurecontainerapps.io"
```

### 8. Deploy Frontend to Static Web Apps

```bash
cd frontend
npm run build

az staticwebapp create \
  --name fastreact-frontend \
  --resource-group fastreact-prod-rg \
  --source . \
  --location eastus2 \
  --branch main \
  --app-location "/" \
  --output-location "build/client"
```

## Environment Variables

Set all required environment variables in the Container App:

```bash
az containerapp update \
  --name fastreact-api \
  --resource-group fastreact-prod-rg \
  --set-env-vars \
    FR_STRIPE_API_KEY="sk_live_..." \
    FR_STRIPE_WEBHOOK_SECRET="whsec_..." \
    FR_RESEND_API_KEY="re_..." \
    FR_GOOGLE_CLIENT_ID="..." \
    FR_GOOGLE_CLIENT_SECRET="..."
```

## CI/CD with GitHub Actions

Add to `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Azure

on:
  push:
    branches: [main]

jobs:
  deploy-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Login to Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Build and push container
        run: |
          az acr build \
            --registry fastreactregistry \
            --image fastreact-api:${{ github.sha }} \
            ./backend

      - name: Deploy to Container Apps
        run: |
          az containerapp update \
            --name fastreact-api \
            --resource-group fastreact-prod-rg \
            --image fastreactregistry.azurecr.io/fastreact-api:${{ github.sha }}
```

## Useful Links

- [Azure Container Apps Docs](https://learn.microsoft.com/en-us/azure/container-apps/)
- [Azure PostgreSQL Flexible Server Docs](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/)
- [Azure Static Web Apps Docs](https://learn.microsoft.com/en-us/azure/static-web-apps/)
