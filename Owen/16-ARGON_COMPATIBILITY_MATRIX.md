# 16 — ARGON COMPATIBILITY MATRIX

**Project:** The Champion — It's All About MU  
**Subsystem:** Development Runtime Demarcation & Google Gemini Argon Capability Mapping  
**Document Ref:** TC-DOC-16-ARGON-MATRIX  
**Authoritative Hierarchy:** Level 2 Specification  

---

## 1. Runtime Demarcation: Argon Dev Environment vs. Production

This matrix explicitly separates what can be developed, simulated, and previewed directly inside the **Google Gemini Argon** environment from what requires external cloud hosting infrastructure.

| Product Requirement | Argon Capability Status | Delivery / Execution Strategy within Argon | External Infrastructure Required |
| :--- | :--- | :--- | :--- |
| **1. Dynamic 4-State UI** | **FULLY SUPPORTED** | Argon generates complete Next.js / React components and can render live interactive previews via `web_code_canvas`. | Cloudflare / Vercel Edge Hosting |
| **2. Sports Engine Core** | **SUPPORTED (Code Gen & Mock)**| Argon generates complete TypeScript normalization logic and mock test suites. | Production Node.js runtime & Sportmonks API key |
| **3. Content Intelligence** | **FULLY SUPPORTED** | Argon natively tests and executes Gemini 1.5 Flash structured JSON prompts, NER logic, and Indonesian synthesis. | Production API Gateway / LiteLLM proxy |
| **4. Multi-Tier Deduplication**| **SUPPORTED (Code Gen & Sim)** | Argon generates MinHash algorithms and PostgreSQL pgvector DDL schema scripts. | PostgreSQL instance with `pgvector` enabled |
| **5. Video Intelligence** | **SUPPORTED (Code Gen & Sim)** | Argon writes YouTube API v3 integration clients, playlist parser logic, and embed validators. | Production YouTube Data API v3 token |
| **6. Workflow Orchestration** | **SUPPORTED (Code Gen & Sim)** | Argon generates BullMQ worker scripts, DAG configurations, and DLQ handlers. | Redis 7 persistent cluster & worker daemon |
| **7. Control Center UI** | **FULLY SUPPORTED** | Complete admin screens, data tables, and review queue interfaces can be generated and rendered. | Production administrative hosting |
| **8. Relational Storage Layer**| **FULLY SUPPORTED (DDL & Migrations)**| Argon generates complete, production-ready PostgreSQL 16 DDL migrations, indexes, and seed files. | Managed PostgreSQL 16 instance |

---

## 2. Definitive Feasibility Declaration

- **Code Generation Feasibility:** 100% of the TypeScript, React, Fastify, Prisma/Kysely, and SQL codebases can be generated directly by Google Gemini Argon.
- **Interactive UI Preview Feasibility:** Complete frontend components and Control Center dashboards can be rendered and previewed via Argon's visual tools (`web_code_canvas`).
- **Production Boundary:** Google Gemini Argon is an agentic development environment and does not serve as an always-on production cloud host (PostgreSQL daemons, Redis clusters, and 24/7 background cron pollers must be deployed to external infrastructure like AWS or DigitalOcean).