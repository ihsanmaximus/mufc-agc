# 03 — Phase 1 Implementation Specification

## Scope
Build the public portal and secure manual CMS. Phase 1 does NOT implement source collection, AI research, AI writing or autonomous publishing.

## Stack
- Next.js App Router, TypeScript
- Modular CSS
- Supabase PostgreSQL, Auth and Storage
- Vercel hosting, GitHub source control

Use stable compatible package versions and document them. Do not add infrastructure without approval.

## Public routes
`/`, `/news`, `/category/[slug]`, `/[categorySlug]/[articleSlug]`, `/search`, `/about`, `/contact`, `/editorial-policy`, `/corrections`, `/privacy-policy`.

## Admin routes
`/admin/login`, `/admin`, `/admin/articles`, `/admin/articles/new`, `/admin/articles/[id]/edit`, `/admin/categories`, `/admin/tags`, `/admin/media`, `/admin/settings`.

## Homepage modules
Header/navigation; featured story; latest news; transfers; match centre; team news; analysis/opinion; popular/recent articles; newsletter placeholder; footer. Hide empty modules or show honest empty states; never invent demo metrics.

## Article page
Title, excerpt, optional approved image, category, byline, published/updated date, body, source attribution links, real related published articles, breadcrumbs and SEO metadata.

## CMS
Admin dashboard; article create/edit/delete; save draft; publish/unpublish; protected preview; category CRUD; tag CRUD; media upload/select; general and SEO settings. Article fields: title, unique slug, excerpt, structured content, category, tags, author, status, featured media, SEO title, meta description, published_at, created_at, updated_at.

## Design
Minimalist editorial style: light neutral background, charcoal text, restrained red accent, sans-serif typography, whitespace and clear hierarchy. Do not copy another publisher's exact design. Mobile-first and accessible. Test 360, 390, 768, 1024 and 1440px. Scope CSS to modules; avoid broad unscoped selectors.

## SEO
Unique title/description, canonical URL from configured base URL, Open Graph, sitemap of published content only, robots.txt blocking private admin/preview paths, accurate Article structured data. Drafts are private and noindex.

## Media
Supabase Storage bucket `mufc-media`; suggested folders `featured/`, `articles/`, `branding/`. Store path, MIME, dimensions, size, alt text, creator/source and rights status. Only approved rights statuses can be used publicly.

## Delivery sequence
A. Initialize repo and Next.js. B. Supabase schema, RLS, storage and auth. C. Public layout/routes. D. Article/category integration. E. Admin auth/authorization. F. Article CRUD and preview/publish. G. Media library. H. SEO. I. QA/deploy.

## Out of scope
RSS collector, scraping, topic clustering, AI research/writer, automatic image retrieval/generation, auto-publisher, advanced analytics, multi-tenant SaaS, payments and newsletter sending. Do not build these in Phase 1.
