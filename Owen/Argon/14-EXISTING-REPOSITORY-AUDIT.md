# 14 — EXISTING REPOSITORY AUDIT & FOUNDATION SEPARATION

**Document Ref:** TC-DOC-14-REPO-AUDIT  
**System:** Technical Audit of Prior Codebases & Foundation Separation  
**Architectural Authority:** Level 2 Audit & Separation Report  

---

## 1. Audit of Prior Codebase (Manchester United News CMS)

### Existing Technical Foundations:
- The previous implementation was conceived as a traditional editorial news CMS (MaxiPress heritage) focused on manual article publishing, category tagging, and standard blog layouts.
- It lacked an authoritative sports engine, possessed no match-context state machine, had no vector deduplication, and treated football news as generic text blogging.

---

## 2. Separation of Foundations: What to Preserve vs. What to Discard

| Existing Component | Audit Verdict | Architectural Action |
| :--- | :--- | :--- |
| **MaxiPress Templates & PHP Code** | **COMPLETELY INCOMPATIBLE** | **Discard 100%.** Zero legacy CMS code is imported into The Champion. |
| **Basic RSS Ingestion Scripts** | **PARTIALLY REUSABLE** | Extract raw feed URLs and HTTP headers; discard extraction logic in favor of `@mozilla/readability`. |
| **Database Schemas (WP/MaxiPress)**| **COMPLETELY INCOMPATIBLE** | **Discard 100%.** Replace with PostgreSQL 16 enterprise schema with `pgvector` (`05-DATABASE-SCHEMA.md`). |
| **Whitelisted Source Feeds** | **REUSABLE** | Import verified Manchester United RSS URLs into `sources` registry table. |
| **Visual Assets (Logos/Colors)** | **REUSABLE** | Adopt Dark Navy (`#0B0E14`) and Manchester Red (`#DA020E`) design tokens (`10-UI-DESIGN-SYSTEM.md`). |

---

## 3. Confirmation of Non-Contradiction

This technical audit confirms that:
1. No legacy architectural constraints from the previous News CMS are dictating The Champion's design.
2. The Champion is built as an independent, automated football fan portal powered by the AGC Engine.
3. All specifications across the 15 Markdown documents in this package are mathematically and architecturally aligned.