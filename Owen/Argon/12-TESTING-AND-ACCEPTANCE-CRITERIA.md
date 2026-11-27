# 12 — TESTING & ACCEPTANCE CRITERIA

**Document Ref:** TC-DOC-12-TESTING  
**System:** Verification Protocols & Quality Assurance Matrix  
**Architectural Authority:** Level 2 Verification Specification  

---

## 1. Testing Framework Matrix

| Tier | Tool / Runner | Target Domain | Coverage Threshold |
| :--- | :--- | :--- | :--- |
| **Unit Tests** | Vitest | Data normalizers, MinHash shingling, confidence formula ($S$) | 100% of mathematical logic |
| **Integration Tests** | Supertest / Testcontainers | Fastify REST endpoints, PostgreSQL queries, Redis BullMQ queues | All API endpoints & migrations |
| **E2E Pipeline Tests**| Custom Pipeline Harness | Full 11-stage AGC processing (Discovery -> Publishing) | 100% of pipeline stages |
| **State Machine Tests**| Vitest Transition Engine | Club match states (`PRE`, `LIVE`, `POST`, `NO_MATCH`) | All 4 states & edge transitions |
| **UI Acceptance Tests**| Playwright | Next.js fan frontend & Control Center screens | Mobile (360px) and Desktop viewports |
| **Load & Stress Tests**| k6 | `/api/v1/layout/homepage` & `/api/v1/matches/live` | 5,000 concurrent Virtual Users |

---

## 2. Core Verifiable Acceptance Criteria

### Test TC-ACC-01: Deterministic Sports Truth
- **Scenario:** An incoming text article reports that Manchester United lost 0-3, but the authoritative sports provider reports a 2-1 victory.
- **Pass Criteria:** The system rejects the article's scoreline; the database `matches` table remains locked to 2-1; zero LLM-generated scorelines overwrite the sports database.

### Test TC-ACC-02: Multi-Tier Deduplication Accuracy
- **Scenario:** Ingest 5 syndicated articles covering the exact same transfer negotiation from different publishers.
- **Pass Criteria:** Exactly 1 article is published as the primary card; the subsequent 4 are linked as attribution references in `story_clusters`; zero duplicate cards appear on the frontend.

### Test TC-ACC-03: Dynamic Homepage State Transition
- **Scenario:** Simulate system time advancing past kickoff into the 68th minute of a live match.
- **Pass Criteria:** Layout API `/api/v1/layout/homepage` automatically shifts `state` from `PRE_MATCH` to `LIVE_MATCH`; Hero section updates from countdown to live scoreboard without requiring manual admin intervention.