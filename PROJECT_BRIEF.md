# MUFC Global Portal — Project Brief

## Vision
Build an independent English-language Manchester United fan publication for a global audience. This portal is the first live niche instance and learning environment for a future reusable AGC SaaS.

## Product separation
- **MUFC Portal:** public website and CMS.
- **AGC Engine:** future system for source collection, topic intelligence, research, article generation, quality control and publishing.

## Locked decisions
- Audience: global.
- Public language: English.
- Design: minimalist news/editorial portal.
- Brand: not selected; do not invent a final name.
- Domain: deferred; use configuration, not hardcoded domain.
- Long-term publishing target: autonomous auto-publishing.
- Current phase: Phase 1 — public portal foundation and manual CMS.
- Phase 1 does not include collector, AI writer or auto-publisher.
- Preferred stack: Next.js, TypeScript, Supabase, Vercel and GitHub.
- Workflow should be manageable from Android browser; do not make Codespaces, Termux or laptop-local development a prerequisite.

## Editorial and legal principles
- This is an independent fan publication, not an official Manchester United website or affiliate.
- Do not use club marks, crests, player photographs or publisher assets without documented permission/licensing.
- Do not copy articles. Use verifiable facts, produce original reporting/context and link/attribute sources.
- Distinguish confirmed facts, attributed reports, rumours, opinion and speculation.
- Never claim exclusive access, official confirmation or endorsement without evidence.
- Provide editorial policy, corrections policy, privacy policy, contact and independence disclosure.

## Engineering principles
Mobile-first, accessible, performant and secure. Supabase is the data source of truth. Drafts must never be public or indexed. Every admin mutation requires server-side authorization and RLS. Never expose service-role keys in client code. Do not add unapproved features.
