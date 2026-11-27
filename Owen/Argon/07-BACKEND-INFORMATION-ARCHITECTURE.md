# 07 — BACKEND INFORMATION ARCHITECTURE & CONTROL CENTER

**Document Ref:** TC-DOC-07-BACKEND-IA  
**System:** The Champion Dedicated Backend & AGC Control Center  
**Architectural Authority:** Level 2 Operational Specification  

---

## 1. Operating Paradigm: Control Center vs. Traditional CMS

The Champion Backend is explicitly designed as an **Automated Operations Control Center**, not a manual word-processing tool:

```
TRADITIONAL CMS PARADIGM:
[Writer] --> [WYSIWYG Editor] --> [Select Categories] --> [Manual Publish Button]

THE CHAMPION CONTROL CENTER PARADIGM:
[Automated Ingestion] --> [Pipeline Processing] --> [Deterministic Scoring S]
                                                             |
                           +---------------------------------+---------------------------------+
                           v                                                                   v
                 (Score S >= 0.88)                                                   (0.65 <= S < 0.88)
                 [Auto-Published to Frontend]                                        [Routed to Review Desk]
```

---

## 2. Information Architecture & Navigation Structure

```
/admin
├── /dashboard                    [System Vitals, Active Match State, 24h Pipeline Throughput]
├── /review-desk                  [Manual Moderation Desk for Borderline & Rumour Content]
│   ├── /pending                  [Pending Items Filtered by S-Score]
│   ├── /flagged-facts            [Items with Entity / Fact Check Discrepancies]
│   └── /transfer-rumours         [Unverified Transfer Speculation]
├── /football-data                [Authoritative Sports Data Monitoring & Overrides]
│   ├── /live-monitor             [Live Match Clock & In-Game Event Inspector]
│   ├── /fixtures                 [Match Schedule, Broadcaster Links & Kickoff Overrides]
│   └── /standings-sync           [League Table Resync Utility]
├── /sources                      [Source Management & Quota Gateway]
│   ├── /registry                 [Whitelisted RSS, Sports API & YouTube Feeds]
│   └── /health                   [Health Monitor, Latency Gauges & Quarantined Sources]
├── /pipeline                     [BullMQ Queue Telemetry & Dead-Letter Queue]
│   ├── /queues                   [Active Job Depths & Worker Concurrency]
│   └── /dlq                      [Dead-Letter Queue Inspector & Replay Utility]
└── /system-health                [Cost Tracking, Token Telemetry & API Usage Limits]
```

---

## 3. Operational Workflows: The Review Desk

Items falling into the moderation window ($0.65 \le S < 0.88$) are presented on the **Review Desk**:

- **Side-by-Side Verification Layout:**
  - *Left Panel:* Cleaned English source text with highlighted named entities and attribution metadata.
  - *Right Panel:* Generated Indonesian title, 3-bullet takeaway, and Indonesian editorial body.
  - *Scoring Banner:* Detailed breakdown of $S$-score ($W_{	ext{source}}, C_{	ext{extractor}}, F_{	ext{fact}}, D_{	ext{dedup}}$).
- **One-Click Actions:**
  - `Approve & Publish`: Transitions item to `published_items` and invalidates CDN cache immediately.
  - `Quick Edit`: Allows in-line correction of Indonesian text before approving.
  - `Reject`: Discards item with operational reason code (`UNRELIABLE`, `FACTUAL_ERROR`, `DUPLICATE`).

---

## 4. Role-Based Access Control (RBAC)

| Role Name | Scope of Permissions |
| :--- | :--- |
| **Super Admin** | Full root access: manage API keys, configure sources, edit worker schedules, purge database partitions, view cost metrics. |
| **Operations Lead** | Review Desk actions, manual live match score overrides, revalidate CDN cache, trigger ad-hoc polling sweeps. |
| **Moderator** | Read-only dashboard access; can Approve, Edit, or Reject items in the Review Desk. |