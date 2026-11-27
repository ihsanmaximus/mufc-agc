# 00 — DOCUMENTATION INDEX & ARCHITECTURAL TAXONOMY

**Project:** The Champion — It's All About MU  
**Engine:** AGC Engine (Automated Content Generation)  
**Document Ref:** TC-DOC-00-INDEX  
**Status:** APPROVED ARCHITECTURAL SPECIFICATION  
**Target Release:** Production v1.0  

---

## 1. Executive Summary & Purpose

This documentation package provides the complete, authoritative technical specifications for **The Champion**, an independent, automated digital platform built for Manchester United fans. Every system, interface, data model, job queue, pipeline stage, and deployment parameter is specified to eliminate architectural ambiguity before any implementation code is written.

All specifications adhere strictly to the non-negotiable core principles:
1. **Independent Stack:** Zero MaxiPress dependencies or legacy CMS paradigms.
2. **Deterministic Sports Facts:** Match scores, statistics, lineups, and standings are sourced strictly from verified sports data APIs; LLMs are forbidden from originating factual sports data.
3. **AGC Operations Engine:** The administrative backend is an operational automation control center, not a conventional editorial publishing tool.
4. **Fan-First Dynamic UX:** Presentation adapts dynamically based on club match lifecycle states (`PRE_MATCH`, `LIVE_MATCH`, `POST_MATCH`, `NO_MATCH`).

---

## 2. Document Hierarchy & System Ownership

The 16 core documentation deliverables are categorized into hierarchical functional tiers. If any inter-document conflict arises, the authoritative hierarchy is established below.

```
                      +------------------------------------------+
                      |       01-PRODUCT_SPECIFICATION.md        |  <-- Business Intent & Functional Scope
                      +--------------------+---------------------+
                                           |
                      +--------------------v---------------------+
                      |       02-SYSTEM_ARCHITECTURE.md          |  <-- Top-Level System Topology & Boundaries
                      +--------------------+---------------------+
                                           |
         +---------------------------------+---------------------------------+
         |                                 |                                 |
+--------v------------------+   +----------v---------------+   +-------------v------------+
| 03-DATABASE_SPECIFICATION |   | 04-AGC_PIPELINE_SPEC     |   | 10-API_AND_DATA_CONTRACTS|
| (Physical Schema & DDL)   |   | (11-Stage Unified DAG)   |   | (REST, WebSocket, Events)|
+--------+------------------+   +----------+---------------+   +-------------+------------+
         |                                 |                                 |
         |         +-----------------------+-----------------------+         |
         |         |                       |                       |         |
+--------v---------v--------+   +----------v---------------+   +---v---------v------------+
| 05-FOOTBALL_DATA_ENGINE   |   | 06-CONTENT_INTEL_SPEC    |   | 07-VIDEO_INTEL_SPEC      |
| (Match Clock & Sports API)|   | (LLM Summaries & Dedup)  |   | (YouTube & Embed Quotas) |
+---------------------------+   +--------------------------+   +--------------------------+
         |                                                                   |
         +---------------------------------+---------------------------------+
                                           |
                      +--------------------v---------------------+
                      |       08-DYNAMIC_PUBLISHING_SPEC         |  <-- 4-State Homepage Composition
                      +--------------------+---------------------+
                                           |
         +---------------------------------+---------------------------------+
         |                                 |                                 |
+--------v------------------+   +----------v---------------+   +-------------v------------+
| 09-CONTROL_CENTER_SPEC    |   | 11-INFRASTRUCTURE_DEPLOY |   | 12-COST_AND_QUOTA_MODEL  |
| (Admin UI & Ops Console)  |   | (Runtime, Redis, Postgres|   | (TCO Scenarios & Quotas) |
+---------------------------+   +----------+---------------+   +--------------------------+
                                           |
         +---------------------------------+---------------------------------+
         |                                 |                                 |
+--------v------------------+   +----------v---------------+   +-------------v------------+
| 13-SECURITY_RELIABILITY   |   | 14-IMPLEMENTATION_ROADMAP|   | 15-TESTING_ACCEPTANCE    |
| (RBAC, Rate Limits, DLQ)  |   | (Phased Execution Plan)  |   | (Verification Framework) |
+---------------------------+   +--------------------------+   +--------------------------+
                                           |
                                +----------v---------------+
                                | 16-ARGON_COMPATIBILITY   |  <-- Development Runtime Demarcation
                                +--------------------------+
```

---

## 3. Master Deliverables Directory

| File Name | Functional Subsystem Covered | Primary Target Audience | Authoritative Scope |
| :--- | :--- | :--- | :--- |
| **01-PRODUCT_SPECIFICATION.md** | Vision, Requirements & Scope | Product Owners, Architects | Business rules, user personas, non-negotiable boundaries, functional matrix. |
| **02-SYSTEM_ARCHITECTURE.md** | Core 8 Subsystems & Data Flow | System Architects, Tech Leads | Service boundaries, communication topologies, high-level Mermaid DAGs. |
| **03-DATABASE_SPECIFICATION.md** | PostgreSQL Physical Schema | DBAs, Backend Engineers | Relational DDL, pgvector integration, indexing, migrations, retention policies. |
| **04-AGC_PIPELINE_SPECIFICATION.md** | 11-Stage Processing Lifecycle | Pipeline & Automation Leads | Unified DAG, error handling, dead-letter logic, idempotency, stage contracts. |
| **05-FOOTBALL_DATA_ENGINE.md** | Sports Provider Normalization | Backend / Data Engineers | Sportmonks/API-Football polling, match clock, live scoring, lineups, standings. |
| **06-CONTENT_INTELLIGENCE_SPEC.md**| Text Scraping & LLM Synthesis | AI / Prompt Engineers | Whitelisted RSS/HTTP, Indonesian translation/synthesis, MinHash & pgvector dedup. |
| **07-VIDEO_INTELLIGENCE_SPEC.md** | YouTube Data API Processing | Integration Engineers | Channel/playlist monitoring, embeddable status validation, daily quota guard. |
| **08-DYNAMIC_PUBLISHING_SPEC.md** | 4-State Homepage Engine | Frontend Architects, UI Devs | State machine (PRE, LIVE, POST, NO_MATCH), layout priority contracts. |
| **09-CONTROL_CENTER_SPEC.md** | Admin Console & Review Queue | UI/UX Engineers, Fullstack Devs | Operational screens, manual review queue, source configuration, overrides. |
| **10-API_AND_DATA_CONTRACTS.md** | API Schemas & Payloads | Frontend & Backend Engineers | OpenAPI 3.0 specs, WebSocket contracts, internal event buses, JSON schemas. |
| **11-INFRASTRUCTURE_AND_DEPLOYMENT.md**| DevOps & Hosting Architecture | SRE, Cloud Engineers | Docker, Kubernetes, AWS/DigitalOcean, Redis 7, Postgres 16, CI/CD pipelines. |
| **12-COST_AND_QUOTA_MODEL.md** | TCO, Quotas & API Budgets | Engineering Managers, CFO | Low/Medium/High capacity forecasts for LLM tokens, Sports APIs, and infrastructure. |
| **13-SECURITY_AND_RELIABILITY.md** | Threat Model & Hardening | Security Engineers, SRE | RBAC, JWT, API key vaulting, sanitization, DLQ retries, disaster recovery. |
| **14-IMPLEMENTATION_ROADMAP.md** | Execution Plan & Phases | Project Managers, Scrum Leads | 6 developmental phases, milestone dependencies, strict Definitions of Done. |
| **15-TESTING_AND_ACCEPTANCE.md** | QA & Test Harnesses | QA Engineers, SDETs | Unit, integration, chaos, state machine, load, and automated end-to-end tests. |
| **16-ARGON_COMPATIBILITY_MATRIX.md**| Argon Dev Environment Limits | All Developers | Environment constraints, native vs external capabilities, runtime boundaries. |

---

## 4. Cross-Document Architectural Traceability

To maintain strict consistency across all documents, standard naming conventions are enforced:
- **Canonical Club Entity:** `club_manchester_united` (UUID or unique string handle).
- **Match Lifecycle Enum:** `PRE_MATCH`, `LIVE_MATCH`, `POST_MATCH`, `NO_MATCH`.
- **Content Classification Enum:** `MATCH_REPORT`, `MATCH_PREVIEW`, `INJURY_UPDATE`, `TRANSFER_CONFIRMED`, `TRANSFER_RUMOUR`, `TACTICAL_ANALYSIS`, `INTERVIEW`, `HISTORIC_REPLAY`.
- **Publishing Status Enum:** `DRAFT`, `PENDING_REVIEW`, `AUTO_PUBLISHED`, `PUBLISHED`, `ARCHIVED`, `REJECTED`.
- **Worker Queues:** `queue:sports:critical`, `queue:content:background`, `queue:video:sync`, `queue:dlq:dead_letter`.

---

## 5. Prerequisite Sign-Off Status

Before initiating code generation or repository scaffolding:
- [x] Technical Audit Review & Acceptance Completed
- [x] 16 Detailed Technical Specification Documents Generated
- [x] Zero Outstanding Inconsistencies Across Subsystems
- [ ] Product Owner Formal Authorization to Begin Phase 1 Implementation Scaffolding