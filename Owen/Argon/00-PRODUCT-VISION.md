# 00 — PRODUCT VISION & FOUNDATIONAL DIRECTIVES

**Document Ref:** TC-DOC-00-VISION  
**System:** The Champion ("It's All About MU")  
**Product Owner:** Kang Ihsan  
**Architectural Authority:** Level 1 Core Blueprint  

---

## 1. Product Mission & Value Proposition

**The Champion** is an independent, automated football fan platform dedicated entirely to Manchester United Football Club.

Traditional digital sports outlets treat football as generic chronological news feeds. Fans are forced to navigate fragmented websites: one for live scores, another for matchday commentary, another for unverified transfer rumours, and scattered social feeds for official highlight clips. 

The Champion unifies the supporter journey into a singular, dynamic, fan-first digital environment powered by an Automated Content Generation (AGC) Engine.

### Core Value Drivers:
- **Football-Contextual Awareness:** The platform dynamically reorganizes its entire presentation based on Manchester United's live match lifecycle (Pre-Match, Live Match, Post-Match, No-Match).
- **Authoritative Sports Integrity:** Match statistics, scores, timelines, and standings are sourced strictly from verified sports feeds with zero hallucination.
- **High-Utility Indonesian Editorial Content:** Transformative AI synthesis converts complex global sports reporting into structured, accessible Indonesian digests with full source provenance and transparent credibility tiers.
- **Official Highlights Hub:** Automated discovery and validation of verified video embeds from official broadcasters and club channels.

---

## 2. Product Boundaries & Non-Negotiables

```
+---------------------------------------------------------------------------------------------------+
|                                  STRICT PRODUCT BOUNDARIES                                        |
+---------------------------------------------------------------------------------------------------+
| [PERMITTED & REQUIRED]                                                                            |
|  - Independent 3-layer stack: AGC Core, The Champion Backend, The Champion Frontend.              |
|  - Purely deterministic sports ingestion (Sportmonks / API-Football).                             |
|  - Transformative, fair-use Indonesian summarization (150-250 words) with source links.           |
|  - Dynamic 4-state homepage orchestration driven by match timing.                                 |
|  - PostgreSQL 16 relational storage with pgvector semantic clustering.                            |
|                                                                                                   |
| [CATEGORICALLY FORBIDDEN]                                                                         |
|  - DO NOT treat the existing News CMS or MaxiPress as the architectural blueprint.                |
|  - DO NOT use LLMs to originate, calculate, or validate match scores, lineups, or standings.      |
|  - DO NOT host, frame, proxy, or redistribute pirated match streams (IPTV, M3U8, illicit embeds). |
|  - DO NOT scrape and republish third-party news articles verbatim.                                |
|  - DO NOT turn this project into a generic multi-niche multi-tenant SaaS before proving MU.        |
+---------------------------------------------------------------------------------------------------+
```

---

## 3. Target User Personas

### Persona A: "Matchday Second-Screener" (Aditya, 26, Bandung)
- **Profile:** Watches every match live via licensed broadcast (Vidio). Uses his smartphone to monitor real-time stats, tactical line-ups, substitutions, and live league table implications.
- **Pain Points:** Mainstream sports portals lag by 3-5 minutes, suffer from slow mobile loading, and drown the screen in intrusive display ads.
- **Platform Solution:** Zero-latency Live Match Center, real-time in-game timeline events, live match clock, and instant post-match official highlights.

### Persona B: "Daily Club Commuter" (Fajar, 34, Jakarta)
- **Profile:** Reads football updates during his morning train commute. Wants fast, verified summaries of club news, transfer negotiations, and tactical breakdowns.
- **Pain Points:** Sensationalist clickbait headlines, misleading transfer rumours, and poor translations of foreign press reports.
- **Platform Solution:** High-confidence Indonesian digests, clear 3-bullet core takeaways, and strict separation between "CONFIRMED" transactions and "SPECULATIVE RUMOURS".

### Persona C: "AGC System Operator" (Automation Manager / Moderator)
- **Profile:** Oversees system health, audits pipeline throughput, configures ingestion sources, and moderates borderline transfer stories.
- **Pain Points:** Traditional CMS editors require tedious manual copying, pasting, formatting, and manual image tagging.
- **Platform Solution:** Automated operations console, real-time queue telemetry, automated confidence scoring ($S$), and a streamlined one-click moderation desk.

---

## 4. Product Success Criteria

1. **Supporter Centricity:** Over 80% of page views land on contextual matchday widgets (Live Center, Countdown, Post-Match Highlights) during fixture windows.
2. **Data Accuracy:** Zero instances of fabricated scores, fictitious kickoff times, or hallucinated player rosters.
3. **Pipeline Velocity:** Raw news stories processed, synthesized, deduplicated, and published within 180 seconds of source publication.
4. **Operational Efficiency:** Less than 10% of ingested content requires manual human intervention, reserved strictly for low-confidence or speculative rumours.