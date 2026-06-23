---
description: "FastReact landing page template — a standalone, SEO-optimized React Router marketing site included with the kit, with built-in Open Graph, Twitter cards, and self-referencing canonicals."
keywords: "fastreact landing page, react router landing template, saas marketing site, seo, open graph, canonical, conversion landing page"
---

# Landing Page Template

FastReact includes a standalone, conversion-focused **landing site** in `landing/` — a separate React Router app from the dashboard (`frontend/`), so your marketing pages deploy independently of the app. It's themed (light/dark) and SEO-optimized out of the box.

## What's included

The home route (`landing/app/routes/home.tsx`) assembles section components from `landing/app/components/landing/`:

- `Topbar`, `Hero`, `Features`, `Testimonial`, `CTA`, `FAQ`, `BundleOffer`, `Footer`

Branding and theming live in `Logo.tsx`, `ThemeToggle.tsx`, and `landing/app/lib/config.ts` (`appName`, `siteUrl`).

## SEO built in

Each route sets metadata through the `seo()` helper (`landing/app/lib/seo.ts`), called from the route's React Router `meta` export. It emits:

- `<title>` and `<meta name="description">`.
- A **self-referencing canonical** auto-derived from the current path — each route points at itself, with no per-page URL to hardcode (override via `canonical` only if needed).
- **Open Graph** tags (`og:type`, `og:url`, `og:title`, `og:description`, `og:site_name`, optional `og:image`).
- **Twitter** card tags (`summary_large_image`, title, description, optional image).

Usage:

```tsx
import { seo } from "~/lib/seo";

export function meta({ location }: Route.MetaArgs) {
  return seo({
    title: "Your Product — tagline",
    description: "One-sentence value proposition.",
    image: "https://yourdomain.com/og-card.png",
  });
}
```

`image` is omitted when unset; the canonical and `og:site_name` come from `config.siteUrl` / `config.appName`. See **[SEO](seo.md)** for the full guidance.

## Customizing

1. Edit the section components in `landing/app/components/landing/` (copy, order, which sections render in `home.tsx`).
2. Set branding in `landing/app/lib/config.ts` and swap `Logo.tsx`.
3. Configure browser-exposed vars in `landing/.env` (`PUBLIC_*` prefix, e.g. `PUBLIC_APP_NAME`, `PUBLIC_API_BASE_URL`).

## Running

See `landing/README.md` for commands (`npm run dev`, `npm run build`). The landing site deploys independently — see [Deployment](../deployment/index.md).

## Next steps

- **[SEO](seo.md)** — sitemaps, robots, and metadata strategy.
- **[Deployment](../deployment/index.md)** — hosting the landing site.
