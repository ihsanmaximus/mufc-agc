# 01 — PRODUCT SPECIFICATION

**Project:** The Champion — It's All About MU  
**Subsystem:** Core Product Specification & Functional Baseline  
**Document Ref:** TC-DOC-01-PROD-SPEC  
**Authoritative Hierarchy:** Level 1 Specification  

---

## 1. Product Vision & Value Proposition

**The Champion** is an independent, automated digital football platform dedicated to Manchester United Football Club. By leveraging an Automated Content Generation (AGC) Engine, the platform eliminates the editorial latency, fragmentation, and overhead of traditional fan media.

### 1.1 Core Value Proposition
- **Real-Time Sports Accuracy:** Millisecond-level updates on matchday fixtures, live scorelines, in-game timelines, and league standings.
- **Context-Aware Presentation:** A dynamic interface that morphs based on Manchester United's live match lifecycle.
- **Localized Content Intelligence:** Transforming verified international reporting into high-quality Indonesian sports analysis while respecting copyright through transformative summarization and provenance citation.
- **Official Highlights Hub:** Automated discovery and validation of official broadcaster embeds.

---

## 2. Target User Personas

### Persona A: "The Live Match Enthusiast" (Budi, 24)
- **Behavior:** Watches matches live; constantly checks second screens for real-time player ratings, tactical substitutions, and live league table shifts.
- **Pain Points:** Cluttered generic news portals; delayed score feeds; intrusive video advertisements.
- **Needs:** Zero-latency live clock, interactive timeline events, instant post-match tactical summaries, and official video replays.

### Persona B: "The Daily Club Commuter" (Rian, 32)
- **Behavior:** Consumes Manchester United news during morning and evening commutes in Jakarta.
- **Pain Points:** Unverified clickbait transfer rumours; poor Indonesian translations of British tabloids.
- **Needs:** Clean, synthesized Indonesian digests of tier-1 verified news; clear distinction between "Rumour" and "Confirmed" transactions.

### Persona C: "The AGC System Operator" (Admin / Automation Lead)
- **Behavior:** Configures whitelist sources, monitors ingestion rate limits, audits dead-letter queues, and verifies flagged articles.
- **Pain Points:** Clunky CMS editors requiring manual copy-pasting of match reports.
- **Needs:** Operational telemetry dashboard, automated confidence threshold sliders, and a streamlined manual review queue.

---

## 3. Product Boundaries & Non-Negotiables

```
+---------------------------------------------------------------------------------------+
|                                PRODUCT INTEGRITY BOUNDARIES                           |
+---------------------------------------------------------------------------------------+
|  [YES] Independent Micro-Frontend & Modular Monolith Backend                           |
|  [YES] PostgreSQL 16 + Redis 7 BullMQ Pipeline Architecture                           |
|  [YES] Strict Source Provenance & Transformation (Indonesian Language Editorial)       |
|  [YES] Authoritative Sports Data Ingestion (Sportmonks / API-Football)                 |
+---------------------------------------------------------------------------------------+
|  [FORBIDDEN] MaxiPress Legacy Code, Templates, Database Schemas, or Admin Paradigms   |
|  [FORBIDDEN] Unauthorized Match Streaming (No Pirated IPTV/M3U8 Feeds)                |
|  [FORBIDDEN] LLM-Generated Match Scores, Dates, Lineups, or Player Records            |
|  [FORBIDDEN] Verbatim Scraping and Redistribution of Copyrighted News Articles        |
+---------------------------------------------------------------------------------------+
```

---

## 4. Functional Requirements

### 4.1 Sports Data Subsystem
- **FR-SD-01:** System must maintain deterministic fixture schedules for Manchester United across all competitions (Premier League, FA Cup, League Cup, UEFA European competitions).
- **FR-SD-02:** Live match mode must ingest scores, clock minutes, and in-game events (goals, cards, substitutions, VAR status) with an update latency under 30 seconds.
- **FR-SD-03:** Official lineups must be ingested and verified 60 minutes prior to scheduled kickoff.
- **FR-SD-04:** Standings must recalculate immediately upon conclusion of relevant league fixtures.

### 4.2 Content Intelligence Subsystem
- **FR-CI-01:** Ingestion of whitelisted RSS feeds and official press releases every 10 minutes (shortened to 5 minutes post-match).
- **FR-CI-02:** AI-driven summarization converting English source text into concise, professional Indonesian briefs (150–250 words).
- **FR-CI-03:** Automated tagging of entities: Players, Manager, Opponent Club, Transfer Category, Matchweek.
- **FR-CI-04:** Multi-tier deduplication enforcing exact URL/text hash, MinHash syntactic similarity, and vector cosine similarity.
- **FR-CI-05:** Transfer intelligence tagging with mandatory classification: `CONFIRMED`, `ADVANCED`, or `RUMOUR`.

### 4.3 Video Intelligence Subsystem
- **FR-VI-01:** Automated discovery of new uploads from official channels (Manchester United, Premier League, Sky Sports, TNT Sports).
- **FR-VI-02:** Embeddable status verification via YouTube API (`status.embeddable == true`).
- **FR-VI-03:** Automatic association of highlight videos with specific `match_id` entities.
- **FR-VI-04:** Fallback UI when highlight embeds are region-restricted or unavailable.

### 4.4 Dynamic Homepage Subsystem
- **FR-DH-01:** Dynamic presentation reacting to four club states: `PRE_MATCH`, `LIVE_MATCH`, `POST_MATCH`, and `NO_MATCH`.
- **FR-DH-02:** Edge-cached payload delivery ensuring page load times under 1.2 seconds globally.

---

## 5. Non-Functional Requirements

| Metric / Dimension | Specification | Verification Method |
| :--- | :--- | :--- |
| **P99 API Latency** | < 150 ms for cached layout endpoints; < 30 ms for Edge CDN hits | Automated load testing (k6) |
| **System Availability** | 99.9% uptime during non-matchday; 99.99% during matchday windows | Synthetic uptime monitoring |
| **Live Match Refresh** | Client polling or WebSocket refresh every 15–30 seconds | End-to-end integration test |
| **Data Freshness** | Sports data < 30s latency; News < 15m from publication | Pipeline latency telemetry |
| **Mobile Responsiveness**| 100% functional parity across mobile, tablet, and desktop (Viewport >= 360px) | Playwright cross-device test |
| **Accessibility & SEO** | Lighthouse score >= 90 for Performance, Accessibility, and SEO | CI Lighthouse audit |

---

## 6. Out-of-Scope Functionality (Strict Exclusions)
1. **User Accounts & Comments:** Version 1.0 is a read-only publication; no public user registrations, social commenting, or chat forums.
2. **E-Commerce & Ticketing:** No direct ticketing sales or merchandise store integration.
3. **Multi-Club Active Deployment:** While the database schema supports multi-tenancy, the active configuration is restricted strictly to Manchester United (`club_id = MU`).