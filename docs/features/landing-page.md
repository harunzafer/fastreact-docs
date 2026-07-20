---
description: "FastReact landing page template — a standalone, SEO-optimized React Router marketing site included with the kit, with built-in Open Graph, Twitter cards, and self-referencing canonicals."
keywords: "fastreact landing page, react router landing template, saas marketing site, seo, open graph, canonical, conversion landing page"
---

# Landing Page Template

FastReact includes a standalone, conversion-focused **landing site** in `landing/` — a separate React Router app from the dashboard (`frontend/`), so your marketing pages deploy independently of the app. It's themed (light/dark) and SEO-optimized out of the box. The steps below take you from the template to your own landing page.

## 1. Find it

The landing site lives in `landing/`. The home route (`landing/app/routes/home.tsx`) assembles section components from `landing/app/components/landing/`. Edit a section to change its content; reorder or remove sections in `home.tsx`.

## 2. Set your branding

- `landing/app/lib/config.ts` — `appName` and `siteUrl`.
- `Logo.tsx` — swap in your logo (light/dark variants).
- `landing/.env` — browser-exposed vars use the `VITE_*` prefix (e.g. `VITE_APP_NAME`, `VITE_SITE_URL`).

## 3. Arrange your sections

`home.tsx` composes the page from these sections, in order:

```tsx
<Topbar />
<Hero />
<Features />
{/* <Showcase /> */}
<Testimonial />
<CTA />
<FAQ />
<Pricing />
<Footer />
```

Reorder, drop, or duplicate any line to change the page. `Showcase` ships commented out — uncomment it (and its import) to enable. `Newsletter` is included as a component but not placed on the page; add `<Newsletter />` (and its import) wherever you want it.

## 4. Write the copy

Each section is its own component in `landing/app/components/landing/`. Edit the one you want:

- **Topbar** — navigation links and the primary CTA.
- **Hero** — headline, subheading, and the main call to action.
- **Features** — the grid of product capabilities.
- **Showcase** — a "copilot in action" demo panel (off by default).
- **Testimonial** — social proof and customer quotes.
- **Pricing** — pricing tiers (this was previously named `BundleOffer`).
- **FAQ** — common questions and answers.
- **CTA** — the closing call to action.
- **Newsletter** — an email-capture signup (UI only — wire the `<form>` to your email provider; off by default).
- **Footer** — links, legal, and social.

## 5. SEO is already handled

Each route sets metadata through the `seo()` helper (`landing/app/lib/seo.ts`), called from its React Router `meta` export, so titles, descriptions, self-referencing canonicals, and Open Graph / Twitter cards are handled for you — no per-page URLs to hardcode. See **[SEO](seo.md)** to customize it.

## 6. Ship it

See `landing/README.md` for commands (`npm run dev`, `npm run build`). The landing site deploys independently — see [Deployment](../deployment/index.md).

## Next steps

- **[SEO](seo.md)** — sitemaps, robots, and metadata strategy.
- **[Deployment](../deployment/index.md)** — hosting the landing site.
