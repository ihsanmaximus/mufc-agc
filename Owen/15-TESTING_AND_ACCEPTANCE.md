# 15 — TESTING AND ACCEPTANCE FRAMEWORK

**Project:** The Champion — It's All About MU  
**Subsystem:** Verification Protocols, Automated Suites & Acceptance Criteria  
**Document Ref:** TC-DOC-15-TEST-ACCEPT  
**Authoritative Hierarchy:** Level 2 Specification  

---

## 1. Testing Framework Matrix

| Testing Tier | Technology / Tool | Target Subsystem | Coverage Mandate |
| :--- | :--- | :--- | :--- |
| **Unit Tests** | Vitest | Data normalizers, MinHash algorithms, confidence scoring formulas | 100% logic coverage |
| **Integration Tests** | Supertest, Testcontainers | Fastify API endpoints, PostgreSQL queries, Redis BullMQ queues | All API routes and transitions |
| **E2E Pipeline Tests** | Custom Harness (Mock API) | 11-stage AGC Pipeline (from raw ingestion to published item) | All 11 stages end-to-end |
| **State Machine Tests**| Vitest State Transition Engine| Club match states (`PRE`, `LIVE`, `POST`, `NO_MATCH`) | 100% transition permutations |
| **UI Acceptance Tests**| Playwright | Next.js fan frontend and Control Center screens | Cross-browser & mobile viewports |
| **Load & Stress Tests** | k6 | `/api/v1/layout/homepage` and `/api/v1/matches/live` | 5,000 concurrent VUs |

---

## 2. Core Verifiable Acceptance Tests

### Test TC-ACC-01: Deterministic Sports Truth
- **Description:** Verify that match results cannot be altered or fabricated by LLM generation.
- **Pass Criteria:**
  - Database rejects any manual or AI score write that does not originate from an authenticated sports provider payload.
  - Scores in `matches` match authoritative provider values with 100% precision.

### Test TC-ACC-02: Multi-Tier Deduplication Accuracy
- **Description:** Inject 5 articles reporting the same transfer rumor from different news outlets.
- **Pass Criteria:**
  - 1st article is processed as the cluster seed.
  - Subsequent 4 articles are identified as duplicates via MinHash or vector cosine distance ($> 0.92$) and merged as source references.
  - Exactly 1 published card appears on the frontend.

### Test TC-ACC-03: Dynamic Homepage State Transition
- **Description:** Simulate fixture clock transitioning through kickoff and full-time.
- **Pass Criteria:**
  - Kickoff $- 48$h: Homepage outputs `PRE_MATCH` payload.
  - Kickoff $0$': Homepage seamlessly switches to `LIVE_MATCH` payload with live scoreboard.
  - Full-Time whistle: Homepage immediately transitions to `POST_MATCH` with highlight video embed.