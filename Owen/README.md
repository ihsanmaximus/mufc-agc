# THE CHAMPION — AGC ENGINE TECHNICAL DOCUMENTATION PACKAGE

**Platform Title:** The Champion  
**Tagline:** It's All About MU  
**Document Release:** Version 1.0 (Production Blueprint)  
**Date:** November 2026  
**Environment Specification:** Google Gemini Argon  

---

## Overview

This repository contains the complete architectural, technical, operational, and database documentation for **The Champion**, an Automated Content Generation (AGC) platform dedicated to Manchester United.

The platform eliminates the latency, manual fragmentation, and editorial overhead of traditional football media by unifying authoritative live sports data, AI-driven contextual news summarization (in Indonesian), official YouTube video replays, and historical club archives into a single dynamic, fan-first frontend experience.

---

## Package Directory Structure

```text
THE-CHAMPION-TECHNICAL-DOCUMENTATION/
├── README.md
└── docs/
    ├── 00-DOCUMENTATION-INDEX.md
    ├── 01-PRODUCT_SPECIFICATION.md
    ├── 02-SYSTEM_ARCHITECTURE.md
    ├── 03-DATABASE_SPECIFICATION.md
    ├── 04-AGC_PIPELINE_SPECIFICATION.md
    ├── 05-FOOTBALL_DATA_ENGINE.md
    ├── 06-CONTENT_INTELLIGENCE_SPEC.md
    ├── 07-VIDEO_INTELLIGENCE_SPEC.md
    ├── 08-DYNAMIC_PUBLISHING_SPEC.md
    ├── 09-CONTROL_CENTER_SPEC.md
    ├── 10-API_AND_DATA_CONTRACTS.md
    ├── 11-INFRASTRUCTURE_AND_DEPLOYMENT.md
    ├── 12-COST_AND_QUOTA_MODEL.md
    ├── 13-SECURITY_AND_RELIABILITY.md
    ├── 14-IMPLEMENTATION_ROADMAP.md
    ├── 15-TESTING_AND_ACCEPTANCE.md
    └── 16-ARGON_COMPATIBILITY_MATRIX.md
```

---

## Critical System Ground Rules

1. **Deterministic Sports Data:** Scores, timers, rosters, statistics, and tables are ingested strictly via authoritative APIs (Sportmonks / API-Football). Large Language Models (LLMs) are categorically barred from originating factual sports facts.
2. **Transformative Editorial AI:** Third-party articles are never scraped and re-published verbatim. The Content Engine extracts raw factual vectors and synthesizes fresh, structured editorial briefs in Indonesian with explicit attribution and links to originators.
3. **No MaxiPress Legacy:** All backend, database, queue, and frontend layers are built from scratch on modern decoupled architectures (Node.js/TypeScript, PostgreSQL 16, Redis 7 BullMQ, Next.js 15).
4. **4-State Dynamic Homepage:** The UI is an active match-state responder (`PRE_MATCH`, `LIVE_MATCH`, `POST_MATCH`, `NO_MATCH`), prioritizing what supporters care about in real time rather than a generic chronological feed.

---

## Verification & Implementation Status

- **Technical Documentation:** 100% Complete & Verified.
- **Application Code:** On hold pending Product Owner sign-off.