# 12 — COST AND QUOTA MODEL

**Project:** The Champion — It's All About MU  
**Subsystem:** Operating Expenses, Token Economics & Provider Quotas  
**Document Ref:** TC-DOC-12-COST-QUOTA  
**Authoritative Hierarchy:** Level 2 Specification  

---

## 1. Operating Cost Scenarios (Monthly TCO)

The infrastructure and API operating expenses are evaluated across three operational load scenarios:

| Expense Category | Low Scenario (MVP / 10k MAU) | Medium Scenario (50k MAU) | High Scenario (250k MAU) | Notes & Pricing Assumptions |
| :--- | :--- | :--- | :--- | :--- |
| **Google Gemini 1.5 Flash API** | $3.50 / mo | $12.00 / mo | $45.00 / mo | Ingesting ~100 articles/day; ~1,500 input + 500 output tokens per run. |
| **Sports API (Sportmonks)** | $35.00 / mo | $35.00 / mo | $75.00 / mo | Standard Pro Football Plan (covers Premier League, FA Cup, Europe). |
| **YouTube Data API v3** | $0.00 / mo | $0.00 / mo | $0.00 / mo | Operates safely within the 10,000 free daily units via playlist polling. |
| **Managed PostgreSQL 16** | $15.00 / mo | $30.00 / mo | $60.00 / mo | DigitalOcean / Supabase managed instance with automated backups. |
| **Managed Redis 7** | $15.00 / mo | $15.00 / mo | $30.00 / mo | Low latency instance for BullMQ queues and edge caches. |
| **Compute & Workers (Docker)**| $12.00 / mo | $24.00 / mo | $48.00 / mo | Container droplet instances running Fastify API and BullMQ workers. |
| **Edge CDN & Hosting (Vercel/CF)**| $0.00 / mo | $20.00 / mo | $40.00 / mo | Cloudflare Pro / Vercel Pro tier for Edge ISR and asset delivery. |
| **Total Monthly Opex** | **~$80.50 / mo** | **~$136.00 / mo** | **~$298.00 / mo** | Well within the $150/mo target budget for baseline operations. |

---

## 2. API Quota Limits & Safety Buffers

```
+-----------------------------------------------------------------------------------+
| PROVIDER       | ALLOCATED DAILY QUOTA | TARGET DAILY CONSUMPTION | BUFFER MARGIN |
+-----------------------------------------------------------------------------------+
| YouTube v3     | 10,000 units          | ~212 units               | 97.8% Safety  |
| Sportmonks v3  | 3,000 calls / hour    | ~240 calls / live hour   | 92.0% Safety  |
| Gemini Flash   | 1,500 RPM (Pay-as-go) | ~10 RPM max peak         | 99.3% Safety  |
+-----------------------------------------------------------------------------------+
```

---

## 3. Cost-Optimization Guardrails

1. **Token Truncation:** Scraped article body text is truncated to a strict 1,200-word limit before passing to Gemini 1.5 Flash, preventing accidental token blowouts.
2. **Strict Playlist Polling:** Disallowing YouTube `search.list` saves 9,788 units per day.
3. **Selective CDN Invalidation:** Utilizing Cloudflare cache tags (`revalidateTag`) ensures that 95% of public visitor page loads are served directly from the edge cache without invoking database read transactions.