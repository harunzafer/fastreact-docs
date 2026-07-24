---
description: "Serve the FastReact app from a sub-path like yourdomain.com/app: two frontend settings, one backend setting, and a list of what stays unchanged."
keywords: "fastreact sub-path, base path, react router basename, vite base, subdirectory deployment, host under path"
---

# Serving the app from a sub-path

The standard setups serve the app from its own subdomain (`app.yourdomain.com`). If you serve it from a path prefix instead, such as `yourdomain.com/app`, three things change: two frontend settings, one backend setting.

## 1. Frontend: set `base` and `basename`

Set both, to the same value, **including the trailing slash**:

```ts
// frontend/vite.config.ts
export default defineConfig({
	base: '/app/',
	// ...
});
```

```ts
// frontend/react-router.config.ts
export default {
	ssr: false,
	basename: '/app/'
} satisfies Config;
```

React Router requires `basename` to begin with Vite's `base` string. With `basename: '/app'` (no trailing slash) the dev server refuses to start, and worse, the production build completes but silently emits no `index.html`.

`<Link>` and `navigate()` pick up the basename automatically. Everything that bypasses the router (hard redirects, image URLs, the Stripe return URL) goes through the helpers in `app/lib/utils/base-path.ts`, which read Vite's base; an eslint rule bans raw `window.location.href` assignments so new code cannot regress this.

## 2. Backend: include the path in `FS_BASE_WEB_URL`

```bash
FS_BASE_WEB_URL=https://yourdomain.com/app
```

No trailing slash here. Every link the backend hands out is built on this value: password reset and verification emails, invitation links, the OAuth redirect after Google sign-in, and the Stripe portal return URL.

## 3. Host: serve the build under the prefix

The app must be reachable under `/app`, and every `/app/*` path that is not a real file must answer with the app's `index.html` (HTTP 200, not a 404 or redirect). This step lives in your hosting setup, and each deployment guide has a **Serving the app from a sub-path** section with the exact change for that stack:

- [Self-Hosting](self-hosting.md#serving-the-app-from-a-sub-path) (nginx location block)
- [Railway](railway.md#serving-the-app-from-a-sub-path) (landing's Caddyfile)
- [DigitalOcean](digitalocean.md#serving-the-app-from-a-sub-path) (component Route setting)
- [Fly.io + Neon + Vercel](fly-neon-vercel.md#serving-the-app-from-a-sub-path) (landing's vercel.json rewrite)
- [Azure](azure.md#serving-the-app-from-a-sub-path) (Front Door path routing)

## What does not change

- **`VITE_API_BASE_URL`**: the backend's absolute URL, independent of where the frontend lives.
- **CORS**: origins are scheme, host, and port. A path prefix plays no role.
- **Cookies**: the session cookie is set with `path=/` and works under any prefix.
- **Google OAuth console**: the authorized redirect URI points at the API, not the app.

## Verify locally

First confirm the build emits the SPA fallback (the silent failure mode of a wrong `basename` is that it does not):

```bash
cd frontend
npm run build
ls build/client/index.html
```

Then run the dev server, which honors the base, and click through login, dashboard, and billing:

```bash
npm run dev
# visit http://localhost:5173/app/login
```
