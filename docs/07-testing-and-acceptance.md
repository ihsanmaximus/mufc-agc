# 07 — Testing and Acceptance

## Responsive QA
Test 360, 390, 768, 1024 and 1440px. Check horizontal overflow, readable typography, image ratios, navigation, touch targets, focus states, card alignment and footer.

## Public tests
- Homepage reads real published Supabase articles.
- Empty sections do not show fake content.
- Category filtering and article slug routing work.
- Anonymous visitors cannot read drafts.
- Search and related articles return published content only.
- Missing image fallback is intentional.
- Canonical, Open Graph and Article metadata are correct.

## Admin tests
- Anonymous and non-admin users cannot access admin mutations.
- Admin can create/edit/save draft and securely preview it.
- Admin can publish/unpublish.
- Published content appears publicly; draft/unpublished content disappears from public routes and sitemap.
- Slug collisions have clear errors.
- Category/tag CRUD works.
- Media type/size validation works.
- Referenced media cannot be accidentally deleted.

## Security tests
RLS verified; no service key in client bundle; rich text sanitized; server actions reject unauthorized users; Storage policies verified; previews private/noindex; secrets absent from logs and repository.

## SEO tests
Unique metadata, canonical correctness, sitemap only published pages, robots blocks private routes, structured data matches visible facts.

## Acceptance
Phase 1 completes only when Vercel deployment works; public portal renders Supabase data; auth/role checks work; article CRUD and draft/publish work; taxonomy and media work; RLS/Storage policies pass; SEO foundation works; responsive QA passes; README and deployment instructions are current.

## Required milestone handover
Report completed tasks, changed files, migrations, required environment variable NAMES (never values), tests and results, deployment URL if any, known issues, deferred work and exact next task.
