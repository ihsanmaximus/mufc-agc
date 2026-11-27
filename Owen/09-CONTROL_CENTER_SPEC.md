# 09 — CONTROL CENTER SPECIFICATION

**Project:** The Champion — It's All About MU  
**Subsystem:** Operational Dashboard, Ingestion Rules & Human Moderation Console  
**Document Ref:** TC-DOC-09-CONTROL-CTR  
**Authoritative Hierarchy:** Level 2 Specification  

---

## 1. Operating Philosophy: Control Center vs. Editorial CMS

The Champion's backend is explicitly designed as an **Automated Content Operations Center**, not an editorial word processor:

```
TRADITIONAL CMS PARADIGM (MaxiPress / WordPress):
[Manual Authoring] --> [Draft Editor] --> [Manual SEO Tagging] --> [Publish Button]

THE CHAMPION AGC CONTROL CENTER PARADIGM:
[Automated Discovery] --> [Pipeline Processing] --> [Deterministic Scoring] 
                                                               │
                                  ┌────────────────────────────┴───────────────────────────┐
                                  ▼                                                        ▼
                        (Score S >= 0.88)                                        (Score 0.65 <= S < 0.88)
                        [Auto-Published]                                         [Surfaced to Review Desk]
```

---

## 2. Core Administrative Screens

### Screen 1: Executive Operations Dashboard (`/admin/dashboard`)
- **System Vitals:** Active match state badge (`PRE`, `LIVE`, `POST`, `NO_MATCH`), worker queue latency, Redis memory footprint, and PostgreSQL connection pool.
- **24-Hour Telemetry:** Ingested raw items count, auto-published percentage, queue exception count, and Gemini API token expenditures.
- **Rapid Actions:** Global cache purge trigger, emergency freeze switch (pauses automated publishing).

### Screen 2: Source & Provider Manager (`/admin/sources`)
- **Table View:** Name, Type (`RSS`, `API`, `YOUTUBE`), Tier (1, 2, 3), Health Status, Last Poll, Success Rate.
- **Configuration Modal:** Set polling frequency, HTTP headers, API keys, and quarantine thresholds.
- **Health Inspector:** View raw response headers, consecutive error logs, and one-click connection testing.

### Screen 3: Review Desk & Moderation Queue (`/admin/review-queue`)
- **Queue Filters:** `PENDING_REVIEW`, `FLAGGED_FACT_CONFLICT`, `TRANSFER_RUMOUR`.
- **Side-by-Side Review Card:**
  - *Left Column:* Extracted English source text with highlighted named entities.
  - *Right Column:* Generated Indonesian title, 3-bullet takeaway, and editorial body.
  - *Scoring Banner:* Displays $S$-score breakdown ($W_{\text{source}}, C_{\text{extractor}}, F_{\text{fact}}, D_{\text{dedup}}$).
- **Operator Actions:**
  - `Approve & Publish`: Commits immediately to `published_items`.
  - `Quick Edit`: Modify Indonesian text directly before publishing.
  - `Reject`: Discards item with operational reason code.

### Screen 4: Football Data & Match Monitor (`/admin/football-data`)
- **Live Match Override:** View current provider feeds, force manual scoreline correction in case of upstream provider lag, and override kickoff times.
- **Lineup Inspector:** View raw lineup payload from Sportmonks and verify player mapping.

### Screen 5: Cost & Telemetry Center (`/admin/cost-center`)
- **API Spend Gauges:** Tracks daily and monthly costs against hard budget limits ($150/mo total budget).
- **Token Usage Breakdown:** Gemini 1.5 Flash input vs. output token graphs.
- **YouTube Quota Gauge:** 0 to 10,000 units tracker with alerts triggering at 8,000 units.

---

## 3. Role-Based Access Control (RBAC)

| Role Name | Access Permissions |
| :--- | :--- |
| **Super Admin** | Full system access: manage API keys, configure sources, edit worker schedules, purge database partitions, view cost metrics. |
| **Operations Lead** | Review Desk actions, manual live match score overrides, revalidate CDN cache, trigger ad-hoc polling sweeps. |
| **Reviewer / Moderator** | Read-only dashboard access; can Approve, Edit, or Reject items in the Review Queue. |