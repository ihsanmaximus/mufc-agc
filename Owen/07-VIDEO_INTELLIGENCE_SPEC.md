# 07 — VIDEO INTELLIGENCE SPECIFICATION

**Project:** The Champion — It's All About MU  
**Subsystem:** YouTube Data API v3 Ingestion, Quota Protection & Embed Management  
**Document Ref:** TC-DOC-07-VIDEO-INTEL  
**Authoritative Hierarchy:** Level 2 Specification  

---

## 1. Official YouTube Whitelist & Channel Directory

The Video Intelligence Engine only processes authorized channels to ensure legal compliance and avoid unauthorized re-streams or clickbait fan vlogs:

| Channel Name | Channel ID | Category | Sync Priority |
| :--- | :--- | :--- | :--- |
| **Manchester United Official** | `UC6yW44UGJJBvYTlfC7CRg2Q` | Official Club Media | P1 (High) |
| **Premier League Official** | `UCG5qGWdu8nIRZqJ_GgDwQ-w` | Competition Highlights | P1 (High) |
| **Sky Sports Football** | `UCNAf1k0yIjyGu3k9BwAg3lg` | Match Highlights & Analysis | P2 (Matchday) |
| **TNT Sports** | `UCrjhj6uUe6F_mR1_HqT1w3A` | European Competition Clips | P2 (Matchday) |
| **MUTV Syndications** | Verified Official Playlists | Interviews & Behind-the-scenes | P3 (Standard) |

---

## 2. API Quota Optimization Model

The default YouTube Data API v3 quota allocation is **10,000 units per rolling 24-hour window**.

### Quota Calculation Table:
- Open Search Endpoint (`search.list`): **100 units per call**.
- Playlist Items Endpoint (`playlistItems.list`): **1 unit per call**.
- Video Details Endpoint (`videos.list`): **1 unit per call**.

> **MANDATE:**  
> The system is strictly forbidden from executing `search.list` queries for standard video discovery. Discovery must operate exclusively via playlist polling (`playlistItems.list`) targeting the "Uploads" playlist ID of whitelisted channels.

```
Uploads Playlist ID Rule:
Replace the 2nd character of the Channel ID with 'U':
Channel ID: UC6yW44UGJJBvYTlfC7CRg2Q
Uploads ID: UU6yW44UGJJBvYTlfC7CRg2Q
```

### Daily Quota Budget:
- 4 Whitelisted Channels $\times$ 24 hourly syncs $\times$ 1 unit = **96 units/day**.
- Post-Match Accelerated Polling (every 10m for 4 hours): 4 channels $\times$ 24 calls $\times$ 1 unit = **96 units**.
- Video Detail Verification (`videos.list` for 20 discovered videos): **20 units**.
- **Total Estimated Quota Consumption:** **~212 units / 10,000 units (2.1% daily utilization)**.

---

## 3. Embed Validation & Geographical Permissions

For every discovered video candidate:
1. Verify `status.embeddable == true`.
2. Inspect `contentDetails.regionRestriction`:
   - If `blocked` list contains `"ID"` (Indonesia), the video is marked `is_embeddable = false`.
   - If `allowed` list is specified and does NOT contain `"ID"`, the video is marked `is_embeddable = false`.
3. Non-embeddable videos are routed to display an external redirection link rather than an embedded player:
   - *"Tonton highlights resmi di YouTube [Tautan Resmi] ->"*

---

## 4. Match-Linking Automation

Videos are linked to authoritative database `matches` via deterministic rule matching:
1. Extract title string and normalize: e.g., `"HIGHLIGHTS | Man Utd vs Liverpool | Premier League"`.
2. Match opponent name against active/recent fixtures within a $\pm 48$-hour window.
3. If title contains `"HIGHLIGHTS"` and matches opponent entity, automatically link `videos.related_match_id = matches.id`.
4. The video is instantly routed to the homepage `WATCH_AGAIN` section.