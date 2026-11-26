# 02 — AGC Technology Specification

## Goal
Create a reusable Automated Generation & Curation engine. MUFC is the first niche configuration, not a hard-coded one-off portal.

## Pipeline
1. Schedule collection from approved sources.
2. Fetch permitted RSS feeds, official feeds/APIs and licensed sources.
3. Parse and normalize source records; store canonical URL, publisher, author, published/fetched timestamps, language, content hash and usage notes.
4. Deduplicate exact and near-duplicate items.
5. Cluster reports into topics/events; preserve disagreements.
6. Extract claims, entities, dates and evidence references.
7. Assess source identity, primary/secondary status, recency and independent corroboration.
8. Generate original explanatory reporting from evidence, not synonym-level rewrites.
9. Generate SEO metadata, category/tag suggestions and relevant image selection.
10. Run mandatory factuality, attribution, originality, rights, privacy, safety and technical gates.
11. Auto-publish only when all mandatory gates pass.
12. Store source relations, model/provider, prompt version, article version, cost and job logs.

## Connector interface
Each source connector should expose `validate_config()`, `fetch_since(cursor, limit)`, `normalize(raw_item)` and `health_check()`.

Prefer sources whose terms permit the intended use. Do not bypass paywalls, authentication, robots restrictions, rate limits or access controls.

## Topic intelligence and evidence
- Normalize player, club, competition and manager names.
- Detect recycled/stale rumours and distinguish publication date from event date.
- Multiple sites repeating one original report are NOT independent corroboration.
- Preserve conflicting claims and uncertainty.
- Store claim-to-source relationships.

## Writer output
Headline, standfirst, structured body, category/tags, SEO title, meta description, source references, image rights state, confidence and quality report. Never invent quotes, fees, statistics, injuries, line-ups or contract details. Clearly distinguish confirmed facts, reported claims, rumours and opinion.

## Image policy
Use owned/licensed/verified public-domain assets or original illustrative artwork. Public availability does not mean reuse permission. Do not scrape image search results or hotlink third-party images. Do not imply an illustration is a real match photograph. If no compliant relevant image exists, publish without one.

## Auto-publish states
`queued -> running -> succeeded`
`running -> retry_wait -> queued`
`running -> needs_research | rejected | failed`

Insufficient/conflicting evidence is held as `needs_research`; rights/policy failure is `rejected`; technical failure is `failed`. Autonomous mode means eligible routine content can publish without human approval, not that uncertain content must be forced live.

## Provider abstraction
Use adapters for LLM, embeddings, image and sports-data providers. Persist provider/model, prompt version, token/cost metadata and timestamps.
