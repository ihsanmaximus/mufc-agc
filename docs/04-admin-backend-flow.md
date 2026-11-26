# 04 — Admin Backend Flow

## Navigation
Dashboard; Articles; Categories; Tags; Media Library; SEO Settings; General Settings.

## Login
1. Open /admin/login.
2. Authenticate with Supabase Auth.
3. Validate session server-side.
4. Check trusted admin/editor role.
5. Allow dashboard access or deny/redirect.
6. Logout invalidates session.
Protect routes AND all server actions/API mutations. Hiding UI is not authorization.

## Dashboard
Show real counts: total articles, published, drafts, categories and recent updates. Provide New Article shortcut. Do not show fake traffic analytics.

## Create article
1. Articles > New Article.
2. Enter title; suggest editable slug.
3. Enter excerpt and body.
4. Select category and tags.
5. Optionally select approved featured image.
6. Enter SEO title and description or accept safe defaults.
7. Save Draft.
8. Server validates authorization and input; persist; return clear success/error feedback.

## Preview
Verify admin session server-side, fetch draft through protected route, render using the public article template. Anonymous visitors must not access draft content. Preview must be noindex and excluded from sitemap.

## Publish
1. Admin clicks Publish and confirms.
2. Server checks role, required fields, slug uniqueness, category and image rights.
3. Set status published.
4. Set published_at on first publication; preserve it during ordinary edits.
5. Make public route and sitemap reflect the article.
6. Return confirmation.

## Unpublish / edit / delete
Unpublish changes status to non-public and removes URL from sitemap. Edits update updated_at without silently resetting original publication date. Destructive deletion requires confirmation; do not delete media still referenced. Prefer archive/soft-delete only if owner approves.

## Categories and tags
Unique normalized slugs. Do not delete a category with articles unless records are reassigned. Prevent duplicate tags and maintain article-tag relations.

## Media
Validate file type/size server-side; upload to Storage; save metadata and rights status; require alt text; check references before delete. Only approved assets may be selected for public articles.

## Settings
General site title/description, logo/favicon and optional base URL; SEO title template and default description. Do not hardcode a final brand or domain.

## Auditability
Record actor ID, action, entity type/ID, timestamp, result and safe metadata for important mutations. Never log passwords, tokens or service keys.

## Future AGC admin menus (NOT Phase 1)
Sources, Collection Monitor, Topics, Research Queue, Generated Articles, Quality Review, Publishing Rules, Automation Schedule, Job Logs, AI Provider Settings, Usage/Cost and Corrections Monitor. Do not create these menus or fake dashboards now.
