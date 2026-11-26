# 01 — System Architecture

## Product boundaries
**MUFC Portal** is the public independent Manchester United fan-news website and its CMS. **AGC Engine** is a separate future system that discovers, researches, generates, validates and publishes content. Do not put AGC pipeline logic in frontend components.

## Phase 1 topology
`Browser -> Next.js (Vercel) -> Supabase PostgreSQL`
`Admin -> Supabase Auth -> protected Next.js server actions/API -> Supabase`
`Media -> Supabase Storage`
`Source control -> GitHub`

## Bounded contexts
1. Public portal: homepage, category, article, search, static policy pages.
2. Admin CMS: authentication, dashboard, articles, categories, tags, media, settings.
3. Content data: articles, taxonomy, media metadata and publication state.
4. Future AGC: collectors, normalization, deduplication, topic intelligence, research, writer, quality gate and publisher.
5. Operations: job queue, retries, logs, monitoring, cost and analytics.

## Future AGC topology
Scheduler -> Source connectors -> Ingestion/normalization -> Deduplication -> Topic clustering -> Evidence/research -> Article generation -> Quality/policy gates -> Publisher -> Portal database -> Monitoring.

## Architecture rules
- Next.js on Vercel; Supabase is persistent source of truth.
- Keep privileged operations server-side. Validate all inputs server-side.
- Public queries return published content only.
- Protect admin pages AND every mutation at the data/action layer.
- Use SQL migrations as schema source of truth.
- Use interfaces/adapters for future AI, source, image and sports-data providers.
- Do not run long-lived multi-step AGC jobs inside one web request.
- Every future job must be idempotent, have a stable ID, bounded retries, timeout, status and structured logs.
- A failed source must not stop other sources. Failed publication retries must not create duplicate articles.
- Never log secrets or credentials.

## Deployment
Use Vercel preview/deployment URL initially. Domain is intentionally deferred. Secrets belong in server-only environment variables.
