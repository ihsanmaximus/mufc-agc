# 13 — ARGON IMPLEMENTATION INSTRUCTIONS

**Document Ref:** TC-DOC-13-ARGON-GUIDE  
**System:** Developer Guidelines & Execution Rules for Gemini Argon  
**Architectural Authority:** Level 2 Developer Instruction Manual  

---

## 1. Mandatory Execution Rules for Gemini Argon

1. **Read All Specifications Before Coding:** Argon must thoroughly study all 15 documentation deliverables before writing application scaffolding.
2. **Strict Module Boundaries:** Argon must follow the 8 logical subsystems and 3-tier layering defined in `01-SYSTEM-ARCHITECTURE.md` and `02-SUBSYSTEM-SPECIFICATIONS.md`.
3. **No Independent Redesign:** Argon is strictly an execution developer. Argon must not simplify requirements, change data contracts, or introduce legacy CMS paradigms.
4. **Phased Milestone Progression:** Implement strictly according to `11-IMPLEMENTATION-ROADMAP.md`. Do not write frontend UI code before the data contracts and database schemas are implemented.
5. **Zero Hallucinated Mock Data in Production:** Argon must use realistic test fixtures matching official sports API schemas. Never hardcode fictitious scores as production data.
6. **Provide Verifiable Evidence:** For every completed milestone, Argon must provide runnable unit/integration tests demonstrating compliance.

---

## 2. Code Organization & Recommended Directory Structure

```text
the-champion/
├── apps/
│   ├── web/                         # Next.js 15 Fan Frontend (Layer 3)
│   │   ├── app/                     # App Router pages (/match, /transfer, /)
│   │   ├── components/              # UI Design System components
│   │   └── hooks/                   # SWR hooks for live match polling
│   └── control-center/              # Next.js 15 AGC Operations Console (Layer 2)
│       ├── app/                     # Admin pages (/dashboard, /review-desk)
│       └── components/              # Telemetry & Review Desk widgets
├── packages/
│   ├── agc-core/                    # AGC Core Engine (Layer 1)
│   │   ├── discovery/               # Source poller & HTTP gateway
│   │   ├── extraction/              # Readability & HTML cleaners
│   │   ├── enrichment/              # Gemini 1.5 Flash prompt harnesses
│   │   ├── dedup/                   # MinHash & pgvector search algorithms
│   │   └── pipeline/                # BullMQ 11-stage DAG coordinators
│   ├── football-engine/             # Authoritative Sports Engine (Layer 2)
│   │   ├── providers/               # Sportmonks & API-Football clients
│   │   ├── state-machine/           # Match state transitions
│   │   └── normalizers/             # Ingestion to canonical sports models
│   ├── database/                    # Shared PostgreSQL 16 Data Layer
│   │   ├── migrations/              # DDL SQL migrations
│   │   └── schema.prisma / kysely   # Type-safe ORM schema
│   └── shared-types/                # Shared TypeScript contracts
│       ├── api.ts                   # REST response contracts
│       └── events.ts                # BullMQ queue event types
```