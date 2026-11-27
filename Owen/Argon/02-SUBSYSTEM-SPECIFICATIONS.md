# 02 — SUBSYSTEM SPECIFICATIONS

**Document Ref:** TC-DOC-02-SUBSYSTEMS  
**System:** The 8 Logical Subsystems of The Champion  
**Architectural Authority:** Level 2 Detailed Functional Specification  

---

## 1. Subsystem A: AGC Engine (Core Orchestration & Automation Infrastructure)

- **Operational Role:** Central job dispatching, workflow coordination, dependency resolution, and automated retry management.
- **Key Modules:**
  - `JobScheduler`: Manages cron intervals and dynamic polling triggers.
  - `QueueManager`: Manages isolated BullMQ queues (`queue:sports:critical`, `queue:content:background`, `queue:video:sync`).
  - `DeadLetterManager`: Captures failed jobs, persists diagnostics, and provides replay capabilities.
- **Interfaces:** Emits job execution events via Redis Pub/Sub; provides programmatic job injection APIs for external ingestion.

---

## 2. Subsystem B: Football Data Engine (Structured Football Facts)

- **Operational Role:** Manages authoritative club entities, fixtures, live scores, lineups, player statistics, and league standings.
- **Key Modules:**
  - `ProviderClient`: Normalizes data from Sportmonks v3 or API-Football.
  - `MatchStateManager`: Manages match status transitions (`SCHEDULED` -> `IN_PLAY` -> `FINISHED`).
  - `EventTimelineSynchronizer`: Ingests and deduplicates goals, substitutions, disciplinary cards, and VAR overturns.
  - `StandingsCalculator`: Computes goal differences, points, and current Premier League rank.
- **Strict Boundary:** 100% deterministic code. LLMs are prohibited from touching this subsystem.

---

## 3. Subsystem C: Source Intelligence (Discovery & Collection Gateway)

- **Operational Role:** Source discovery, credibility assessment, raw collection, and HTTP/RSS parsing.
- **Key Modules:**
  - `SourceRegistry`: Manages authorized source URLs, polling frequencies, and credibility tiers (Tier 1: Official/Tier 1, Tier 2: Reliable, Tier 3: Speculative).
  - `HealthMonitor`: Monitors HTTP status codes, consecutive timeouts, and manages automated quarantine states.
  - `PayloadHarvester`: Fetches feeds, calculates SHA-256 hashes, and writes immutably to `raw_items`.

---

## 4. Subsystem D: Content Intelligence (Analysis & Editorial Processing)

- **Operational Role:** Article extraction, Named Entity Recognition (NER), multi-tier deduplication, and transformative Indonesian editorial generation.
- **Key Modules:**
  - `ReadabilityExtractor`: Extracts article titles, bylines, dates, and clean body text while discarding advertising boilerplate.
  - `EntityRecognizer`: Identifies player names, managerial staff, competitions, and rival clubs.
  - `MultiTierDeduplicator`: Executes exact MD5, syntactic MinHash, and semantic pgvector cosine distance checks.
  - `EditorialSynthesizer`: Prompts Gemini 1.5 Flash at temperature 0.2 to generate punchy Indonesian titles, 3-bullet takeaways, and balanced narratives with source links.

---

## 5. Subsystem E: Video Intelligence (Discovery & Embed Management)

- **Operational Role:** Discovers official Manchester United video content, checks embed permissions, and links clips to matches.
- **Key Modules:**
  - `YouTubePlaylistPoller`: Targets "Uploads" playlists of verified channels (Official MU, Premier League, Sky Sports) using low-cost `playlistItems.list` (1 quota unit).
  - `EmbedPermissionValidator`: Verifies `status.embeddable == true` and verifies territorial rights for Indonesia (`ID`).
  - `MatchVideoLinker`: Correlates video titles and timestamps against recent fixtures in the `matches` table.

---

## 6. Subsystem F: Visual Intelligence (Image Discovery & Asset Management)

- **Operational Role:** Image discovery, relevance validation, aspect ratio standardization, and responsive asset delivery.
- **Key Modules:**
  - `VisualAssetValidator`: Ensures image URLs are responsive, minimum resolution (1200x675 for 16:9 hero), and within allowed MIME types.
  - `ThumbnailOptimizer`: Generates WebP variations (mobile, desktop, thumbnail) and stores metadata.
  - `FallbackVisualProvider`: Provides verified club fallback imagery (Old Trafford, official crest, team celebration) when source images are missing or copyright-restricted.

---

## 7. Subsystem G: Dynamic Publishing Engine (Homepage & Layout Composition)

- **Operational Role:** Dynamic layout composition, state evaluation, section sequencing, and edge cache invalidation.
- **Key Modules:**
  - `MatchContextEvaluator`: Determines active club state: `PRE_MATCH`, `LIVE_MATCH`, `POST_MATCH`, or `NO_MATCH`.
  - `LayoutComposer`: Assembles the unified dynamic JSON layout contract for the frontend.
  - `CacheTagPurger`: Triggers edge cache invalidations (`revalidateTag('homepage')`) upon match score shifts or breaking news publication.

---

## 8. Subsystem H: Control Center (Operational Console & Review Desk)

- **Operational Role:** Administrative monitoring, manual moderation, pipeline telemetry, and configuration management.
- **Key Modules:**
  - `OperationsDashboard`: Displays active club state, queue latencies, and token expenditures.
  - `ReviewDesk`: Side-by-side interface for operators to review, edit, approve, or reject low-confidence items and transfer rumours.
  - `SourceConfigurator`: UI to adjust source tiers, polling schedules, and trigger manual ingestion sweeps.