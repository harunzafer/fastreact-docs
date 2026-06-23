---
description: "Add SEO metadata to your FastReact landing pages - use the bundled seo() helper for titles, meta descriptions, self-referencing canonicals, and Open Graph / Twitter cards on every route."
keywords: "fastreact seo, react router meta, canonical url, open graph, twitter cards, meta description, react router 7 seo, page metadata, react seo"
---

# SEO & Page Metadata

[FastReact](https://fastreact.dev)'s landing ships with a reusable `seo()` helper
(`app/lib/seo.ts`) that builds a correct `<head>` for any route: a single title and meta
description, a **self-referencing canonical**, and Open Graph / Twitter card tags. The homepage is
already wired up — you only need this guide when you **add new routes**.

## Using the helper

React Router 7 emits per-route head tags through each route module's [`meta`
export](https://reactrouter.com/start/framework/route-module#meta), which the `<Meta />` component
in `app/root.tsx` renders. Call `seo()` from that export and pass the current path:

```tsx
import type { Route } from './+types/pricing';
import { seo } from '~/lib/seo';

export function meta({ location }: Route.MetaArgs) {
	return seo({
		title: 'NoteAI — Pricing',
		description: 'Simple, transparent pricing for every stage of your SaaS.',
		pathname: location.pathname
	});
}
```

The `location` argument supplies `pathname`, which the helper uses to derive the canonical — so
every route points at itself with nothing to hardcode.

### Arguments

| Field           | Required | Description                                                                       |
| --------------- | -------- | --------------------------------------------------------------------------------- |
| `title`         | yes      | Page `<title>` and default social title.                                          |
| `description`   | yes      | `<meta name="description">` and default social description.                        |
| `pathname`      | yes      | Current path (use `location.pathname`); drives the self-referencing canonical.    |
| `ogTitle`       | no       | Override the social title (defaults to `title`).                                  |
| `ogDescription` | no       | Override the social description (defaults to `description`).                      |
| `image`         | no       | Absolute URL to a social-card image. When omitted, no `og:image` tag is emitted.  |
| `canonical`     | no       | Override the auto-derived canonical (rarely needed).                              |

## Why a shared helper (and not a layout default)

It's tempting to put a default description or canonical somewhere global — e.g. the `meta` export in
`root.tsx` — so every route inherits it. **Don't.** React Router merges meta by leaf route, but a
global default still ships on pages that don't override it, and two things go wrong:

- **Duplicate / stale descriptions.** A page that sets its own description on top of a global default
  can end up with two `<meta name="description">` tags, or silently inherit the wrong one. (A page
  with no description of its own ships just the default — the trap springs once a page tries to
  override it, which most will.)
- **Leaked canonical.** A hardcoded homepage canonical inherited by every route is a *consolidation
  hint* telling search engines those pages are duplicates of the homepage, so their ranking signals
  get folded into it and they don't rank as themselves. (It's a hint, not a `noindex` — search
  engines may ignore it — but you don't want to be fighting it.)

`seo()` avoids both by returning exactly one description and a canonical derived from the **current
path**:

```
/            → https://yourdomain.com/
/pricing     → https://yourdomain.com/pricing
```

## Configure your domain

Canonical and `og:url` tags are built from `config.siteUrl`, which reads the `VITE_SITE_URL`
environment variable (see `app/lib/config.ts`). Set this to your own domain so the tags don't point
at `fastreact.dev`:

```bash
# .env
VITE_SITE_URL=https://yourdomain.com
```

`og:site_name` uses `config.appName` (`VITE_APP_NAME`), so set that too if you've renamed your app.

## Social card images

The helper omits `og:image` / `twitter:image` unless you pass an `image`. To enable rich social
previews, add a `1200×630` PNG/JPG to `public/` and pass its absolute URL:

```tsx
return seo({
	title: 'NoteAI — Pricing',
	description: 'Simple, transparent pricing for every stage of your SaaS.',
	pathname: location.pathname,
	image: `${config.siteUrl}/images/og-image.png`
});
```

## Verify

After `npm run build` and `npm run start`, inspect the homepage source. You should see exactly
**one** `<meta name="description">` and **one** self-referencing `<link rel="canonical">`:

```bash
curl -s http://localhost:3000/ | grep -E 'name="description"|rel="canonical"'
```
