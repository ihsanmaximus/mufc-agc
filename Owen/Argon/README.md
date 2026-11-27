# THE CHAMPION — AGC FAN PORTAL
## Architecture Documentation & Argon Implementation Specification Package

**Product:** The Champion ("It's All About MU")  
**Core Engine:** AGC Core Engine (Automated Content Generation)  
**System Architect:** Zaki  
**Product Architect / Technical Auditor:** Owen  
**Developer:** Gemini Argon  
**Product Owner:** Kang Ihsan  
**Document Package Version:** 1.0.0-RELEASE  
**Date:** November 2026  

---

## 1. Executive Summary & Purpose

This package contains the complete, authoritative, and unambiguous technical specification package for **The Champion**, an independent Manchester United football fan portal powered by the shared AGC (Automated Content Generation) Engine.

### Fundamental Project Directives:
1. **Dedicated Fan Platform, Not a News CMS:** The Champion is engineered around football context, supporter matchday rituals, authoritative sports data, and curated multimedia—not chronological article blogging.
2. **Three-Layer Architecture:**
   - **Layer 1: AGC Core Engine** — Reusable automation, discovery, normalization, enrichment, deduplication, and publishing orchestration.
   - **Layer 2: The Champion Application** — Dedicated football domain logic, match state machine, editorial review desk, provider integrations, and fan-facing backend.
   - **Layer 3: The Champion Frontend** — Public-facing responsive supporter experience dynamically shaped by match timing and club context.
3. **Deterministic Sports Truth:** Live scores, fixtures, lineups, match clocks, standings, and match events are strictly ingested from authoritative sports data providers. Large Language Models (LLMs) are categorically barred from originating, guessing, or validating factual sports data.
4. **Transformative Content Intelligence:** Third-party journalism is never scraped and republished verbatim. The engine synthesizes original, structured Indonesian editorial summaries with strict provenance attribution and direct source links.
5. **No Legacy MaxiPress Inheritance:** Zero code, templates, or legacy CMS assumptions from MaxiPress or previous news portals are carried over.

---

## 2. Document Hierarchy & Implementation Roadmap

The package consists of 15 authoritative Markdown documents structured in logical implementation order:

```
THE-CHAMPION-SPECIFICATION-PACKAGE/
├── README.md                                  [Package Overview & Master Index]
├── 00-PRODUCT-VISION.md                       [Vision, Principles, Personas & Boundaries]
├── 01-SYSTEM-ARCHITECTURE.md                  [3-Tier Layering, 8 Subsystems & Mermaid Topologies]
├── 02-SUBSYSTEM-SPECIFICATIONS.md             [Detailed Specifications of 8 Logical Subsystems]
├── 03-AUTOMATION-PIPELINE.md                  [11-Stage Unified DAG Lifecycle & Retry Policies]
├── 04-DATA-ARCHITECTURE.md                    [Data Classification, Lineage, Provenance & Retention]
├── 05-DATABASE-SCHEMA.md                      [PostgreSQL 16 Physical DDL & pgvector Setup]
├── 06-API-AND-MODULE-CONTRACTS.md             [REST Endpoints, BullMQ Events & JSON Schemas]
├── 07-BACKEND-INFORMATION-ARCHITECTURE.md     [AGC Control Center, Navigation & Review Desk]
├── 08-FRONTEND-EXPERIENCE.md                  [Fan-First UI, Responsive Design & Mobile Behavior]
├── 09-DYNAMIC-HOMEPAGE-ENGINE.md              [4-State Match Engine: Pre, Live, Post, No-Match]
├── 10-UI-DESIGN-SYSTEM.md                     [Design Tokens, Dark Navy/Red Palette & Components]
├── 11-IMPLEMENTATION-ROADMAP.md               [6 Milestones, Dependencies & Definitions of Done]
├── 12-TESTING-AND-ACCEPTANCE-CRITERIA.md      [QA Harness, Verification Tests & Acceptance Matrix]
├── 13-ARGON-IMPLEMENTATION-INSTRUCTIONS.md    [Execution Rules, Code Scaffolding for Gemini Argon]
└── 14-EXISTING-REPOSITORY-AUDIT.md            [Audit of Prior Codebases & Foundation Separation]
```

---

## 3. Implementation Sequence for Gemini Argon

Gemini Argon must strictly follow this phased roadmap. **No milestone may commence until the prior milestone's acceptance criteria are tested and verified:**

1. **Milestone 1:** Repository Scaffolding, TypeScript Contracts & PostgreSQL 16 DDL Migration (`05-DATABASE-SCHEMA.md`, `06-API-AND-MODULE-CONTRACTS.md`).
2. **Milestone 2:** Football Data Engine & Authoritative Sports Provider Synchronization (`02-SUBSYSTEM-SPECIFICATIONS.md`, `09-DYNAMIC-HOMEPAGE-ENGINE.md`).
3. **Milestone 3:** Source Gateway, Extraction & 11-Stage AGC Content Pipeline (`03-AUTOMATION-PIPELINE.md`, `04-DATA-ARCHITECTURE.md`).
4. **Milestone 4:** Video & Visual Intelligence Subsystems (`02-SUBSYSTEM-SPECIFICATIONS.md`).
5. **Milestone 5:** Dynamic 4-State Fan Frontend (`08-FRONTEND-EXPERIENCE.md`, `09-DYNAMIC-HOMEPAGE-ENGINE.md`, `10-UI-DESIGN-SYSTEM.md`).
6. **Milestone 6:** AGC Control Center, Review Desk & Production Hardening (`07-BACKEND-INFORMATION-ARCHITECTURE.md`, `12-TESTING-AND-ACCEPTANCE-CRITERIA.md`).

---

## 4. Implementation Rules & Sign-Off Notice

- **No Unauthorized Architecture Redesign:** Argon is an execution developer and must strictly adhere to the module boundaries and contracts defined in this specification package.
- **Evidence-Based Completion:** Every completion claim must be backed by unit, integration, or E2E test passes.
- **Explicit PO Sign-Off:** Implementation begins only upon formal approval of this documentation package by Product Owner Kang Ihsan.