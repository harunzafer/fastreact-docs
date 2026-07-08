---
description: "FastReact troubleshooting guide - Solutions for common setup, database, authentication, and deployment issues with FastAPI, React Router v7, PostgreSQL, and Docker."
keywords: "fastreact troubleshooting, fastapi errors, react router v7 issues, postgresql problems, docker debugging, database migration errors, authentication issues, cors errors, deployment problems"
---

# Troubleshooting

Common issues and solutions for [FastReact](https://fastreact.dev) development and deployment.

## Setup Issues

**Init script prerequisites not met**

```bash
✗ Python 3.12 not found
```

**Solution:** Install all required tools:

- **Python 3.12+**: https://www.python.org/downloads/
- **Docker**: https://docs.docker.com/get-docker/
- **Node.js 22+**: https://nodejs.org/

**Init script permission denied**

```bash
bash: ./init.py: Permission denied
```

**Solution:** Make the script executable:

```bash
chmod +x init.py
chmod +x backend/db/sqitch.sh
```

**Port conflicts during init**

```bash
Error: address already in use :::5432
```

**Solution:** Stop conflicting services:

```bash
# Find what's using port 5432
lsof -i :5432

# Stop the service or change the port in docker-compose.yml
```

---

## Database Issues

**Migration fails with "relation does not exist"**

The schema hasn't been created yet. Run migrations in order:

```bash
cd backend/db
./sqitch.sh dev deploy
```

**"could not connect to server"**

The database container isn't running:

```bash
docker compose up db -d
```

**Migration fails mid-way**

Check which migrations are deployed:

```bash
cd backend/db
./sqitch.sh dev status
```

Revert to a safe point and redeploy:

```bash
./sqitch.sh dev revert --to migration_name
./sqitch.sh dev deploy
```

---

## Authentication Issues

**Login returns 401 with correct credentials**

1. Verify your admin user was created: `uv run scripts/create_admin.py`
2. Check that `FR_ENVIRONMENT` is set correctly in `backend/.env`
3. Verify database is running and migrations are applied

**"CORS error" in browser console**

Add your frontend URL to CORS settings in `backend/.env`:

```bash
FR_CORS_ORIGINS="http://localhost:5173,http://localhost:4173"
```

**Google OAuth redirect fails**

Ensure your redirect URI in Google Cloud Console exactly matches:

```
http://localhost:8000/auth/oauth/google/callback
```

**Session cookie not being set**

In development, ensure `FR_ENVIRONMENT=dev` is set. The cookie `SameSite` and `Secure` settings differ between dev and production.

---

## Frontend Issues

**API client types are out of sync**

Regenerate the TypeScript client after backend changes:

```bash
cd frontend
npm run generate
```

**TypeScript errors after `npm run generate`**

Backend changed a model that the frontend uses. Check the TypeScript errors to find what changed, then update your components accordingly.

**Vite build fails**

```bash
npm run typecheck
```

Fix all TypeScript errors, then retry the build.

**React Router v7 route not found**

Ensure the route is registered in `frontend/app/routes.ts`:

```typescript
route("my-page", "routes/protected/my-page.tsx"),
```

---

## Backend Issues

**`ModuleNotFoundError` after adding code**

The dependency injection container hasn't been updated. Add your new service/repo to `app/config/container.py` and wire it up.

**`422 Unprocessable Entity` from API**

The request body doesn't match the Pydantic model. Check:

1. Required fields are present
2. Field types match (e.g., sending a string where an int is expected)
3. Visit `http://localhost:8000/docs` to see the expected schema

**`500 Internal Server Error`**

Check the backend terminal for the full Python traceback. Common causes:

- Database connection failed
- Missing environment variable
- Unhandled exception in a service

**`409 Conflict` on user creation**

The email already exists in the database. Use a different email or check the existing user.

---

## Email Issues

**Emails not received in development**

By default, development uses `stub` mode — check your backend logs for `[STUB EMAIL]` blocks containing the email content and links.

**Email not sending in production**

1. Verify `FR_EMAIL_PROVIDER` is set correctly
2. Check your API key is valid
3. Ensure your sender domain is verified with the provider
4. Check provider rate limits

---

## Stripe Issues

**Webhook signature verification fails**

The `FR_STRIPE_WEBHOOK_SECRET` doesn't match the one configured in Stripe dashboard.

Use the Stripe CLI for local testing:

```bash
stripe listen --forward-to localhost:8000/webhook/stripe
```

The CLI prints a webhook signing secret — use that value for local development.

**Subscription not updating after payment**

Webhooks aren't reaching your server. Check:

1. Stripe CLI is running (development) or webhook endpoint is configured (production)
2. The endpoint returns a 200 response
3. Check Stripe dashboard webhook logs for delivery attempts

---

## Docker Issues

**Container exits immediately**

Check logs:

```bash
docker compose logs api
```

**Database volume conflicts**

Reset the database volume:

```bash
docker compose down -v
docker compose up db -d
cd backend/db && ./sqitch.sh dev deploy
```

---

## Getting More Help

If you're stuck:

1. Check the [GitHub Issues](https://github.com/harunzafer/fastreact/issues) for similar problems
2. Search for the error message online
3. Review the [Architecture Overview](architecture.md) to understand expected behavior
4. Enable debug logging in `backend/.env`: `FR_LOG_LEVEL=debug`
