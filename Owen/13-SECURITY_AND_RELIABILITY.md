# 13 — SECURITY AND RELIABILITY

**Project:** The Champion — It's All About MU  
**Subsystem:** Threat Mitigation, Zero-Trust Hardening & Reliability Engineering  
**Document Ref:** TC-DOC-13-SEC-RELIABILITY  
**Authoritative Hierarchy:** Level 2 Specification  

---

## 1. Threat Matrix & Architectural Safeguards

| Threat Vector | Attack Scenario | Architectural Defense Mechanism |
| :--- | :--- | :--- |
| **Unauthorized Match Streams** | User or scraper attempts to inject pirated IPTV/M3U8 links into match center. | **Strict Ingestion Schema:** Match links are only accepted from verified official broadcast partners (Vidio ID, Sky, MUTV) hardcoded in the database. |
| **Prompt Injection / Jailbreak** | Malicious text in a scraped blog attempts to compromise the summary model. | **Isolated Context Delimitation:** Raw text is inserted into the prompt inside strict XML tags (`<source_text>...</source_text>`) with explicit system instructions to treat content strictly as inert data. |
| **API Denial of Service (DoS)** | Malicious traffic floods live match endpoints. | **Cloudflare WAF Rate Limiting:** Enforces maximum 60 requests/minute per IP; edge cache absorbs 95% of traffic. |
| **Credential Exfiltration** | Database or worker logs leak API credentials. | **Secret Zero Policy:** All credentials vaulted in environment variables; structured logging libraries strictly redact `authorization`, `password`, and `token` keys. |

---

## 2. Dead-Letter Queue (DLQ) & Failure Recovery

```
[BullMQ Worker Job]
        │
        ├──> Success ──> [Commit to Database]
        │
        └──> Failure (Exception / Timeout)
                 │
                 ├──> Attempts < 3 ──> [Exponential Backoff Retry] (1s, 5s, 15s)
                 │
                 └──> Attempts == 3 ──> [Route to DLQ: queue:dlq:dead_letter]
                                                │
                                                ▼
                                    [Alert Emitted to Control Center]
```

### DLQ Ingestion Rules:
- When a job exhausts its 3 retries, the entire context (job payload, error stack trace, attempt counts, execution duration) is serialized into `pipeline_job_logs` and held in `queue:dlq:dead_letter`.
- Admin Control Center provides a one-click "Inspect & Re-queue" button for transient upstream provider outages.