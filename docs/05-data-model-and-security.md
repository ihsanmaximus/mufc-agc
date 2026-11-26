# 05 — Data Model and Security

## Core schema
- `categories`: id UUID, name, unique slug, description, created_at.
- `articles`: id UUID, title, unique slug, excerpt, content (prefer JSONB structured editor), category_id FK, author_name, status (draft/published), featured_media_id FK, seo_title, meta_description, published_at, created_at, updated_at.
- `tags`: id UUID, name, unique slug, created_at.
- `article_tags`: article_id FK, tag_id FK, composite primary key.
- `media_assets`: id, bucket, storage_path, mime_type, file_size, width, height, alt_text, caption, creator, source_url, rights_status, uploaded_by, created_at.
- `site_settings`: site_name, description, logo/favicon, default SEO title/description, updated_at.
- `profiles` (if needed): auth user ID, trusted role (admin/editor), display_name.
- `audit_logs`: actor, action, entity type/ID, result, safe metadata, created_at.

Rights statuses: owned, licensed, public_domain_verified, generated_original, pending, prohibited. Only approved statuses may be used publicly.

## Security
- Enable RLS on every exposed table.
- Anonymous users can read published articles and public taxonomy only.
- Anonymous users cannot insert/update/delete.
- Admin/editor permissions derive from trusted server-side role mapping, never client-submitted role values.
- Protect Storage upload/delete with policies.
- Validate MIME type, size and content server-side.
- Sanitize rich content before rendering.
- Prevent XSS, SQL injection and open redirects.
- Rate-limit auth and sensitive endpoints where possible.
- Keep service-role and AI keys server-only; never expose in browser bundle.
- Do not weaken RLS/auth to make a feature work.

## Integrity
Unique normalized slugs; published articles require title/body and required taxonomy; drafts may be incomplete. Preserve first published_at unless explicit republish. Do not delete referenced media. Use transactions for related writes where possible. All schema changes go through committed SQL migrations.
