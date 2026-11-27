# 10 — API AND DATA CONTRACTS

**Project:** The Champion — It's All About MU  
**Subsystem:** Internal REST Endpoints, Event Schemas & Live Synchronization  
**Document Ref:** TC-DOC-10-API-CONTRACTS  
**Authoritative Hierarchy:** Level 2 Specification  

---

## 1. REST API Standards & Base Conventions

- **Base URL:** `https://api.thechampion.club/api/v1`
- **Content Negotiation:** `application/json; charset=utf-8`
- **Authentication:**
  - Public Client Endpoints: Edge rate-limited (Cloudflare).
  - Admin / Control Center Endpoints: `Bearer <JWT_TOKEN>` with RS256 signing.
- **Standard Error Envelope:**
```json
{
  "status": "error",
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Match entity with specified ID does not exist.",
    "details": { "requested_id": "9a12c8b0-8f12" },
    "timestamp": "2026-11-27T07:45:00Z"
  }
}
```

---

## 2. Core API Endpoints

### 2.1 Dynamic Homepage Layout
- **Endpoint:** `GET /layout/homepage`
- **Cache Strategy:** Edge cached with `Cache-Control: public, s-maxage=15, stale-while-revalidate=45`.
- **Response Structure:** Defined completely in `08-DYNAMIC_PUBLISHING_SPEC.md`.

### 2.2 Live Match Status & Timeline
- **Endpoint:** `GET /matches/live`
- **Description:** Returns real-time scorecard, minute clock, and event stream for the active match.
- **Response:**
```json
{
  "status": "success",
  "data": {
    "match_id": "9a12c8b0-8f12-4211-b841-f76192837411",
    "status": "IN_PLAY",
    "period": "2H",
    "minute": 68,
    "scores": { "home": 2, "away": 1 },
    "teams": {
      "home": { "name": "Manchester United", "short": "MUN", "crest": "https://cdn.thechampion.club/crests/mun.png" },
      "away": { "name": "Liverpool", "short": "LIV", "crest": "https://cdn.thechampion.club/crests/liv.png" }
    },
    "events": [
      { "minute": 24, "type": "GOAL", "player": "Bruno Fernandes", "team": "MUN", "detail": { "assist": "Marcus Rashford" } },
      { "minute": 41, "type": "GOAL", "player": "Rasmus Højlund", "team": "MUN", "detail": {} },
      { "minute": 58, "type": "GOAL", "player": "Mohamed Salah", "team": "LIV", "detail": {} }
    ],
    "statistics": {
      "possession": { "home": 48, "away": 52 },
      "shots_on_target": { "home": 6, "away": 4 },
      "xg": { "home": 1.84, "away": 1.21 }
    }
  }
}
```

### 2.3 News & Transfer Intelligence Feed
- **Endpoint:** `GET /content/feed`
- **Query Params:** `category` (`ALL`, `TRANSFERS`, `MATCHES`), `limit` (default: 20), `cursor` (pagination).
- **Response:**
```json
{
  "status": "success",
  "data": {
    "items": [
      {
        "id": "c1092834-4123",
        "title": "United Capai Kesepakatan Personal dengan Bek Tangguh",
        "category": "TRANSFER_RUMOUR",
        "transfer_badge": "RUMOUR",
        "summary_bullets": [
          "Kesepakatan kontrak berdurasi lima tahun telah disetujui",
          "Klub kini memulai negosiasi biaya transfer resmi",
          "Pemain diproyeksikan mengisi bek tengah utama musim depan"
        ],
        "published_at": "2026-11-27T06:30:00Z",
        "provenance": {
          "source_name": "Fabrizio Romano",
          "source_url": "https://twitter.com/example"
        }
      }
    ],
    "next_cursor": "eyJjcmVhdGVkX2F0IjoxNzM..."
  }
}
```

---

## 3. Internal Event Bus Contracts (Redis Pub/Sub)

When pipeline workers complete critical tasks, events are broadcast internally:

| Event Name | Producer | Payload Keys | Consumers |
| :--- | :--- | :--- | :--- |
| `match.score_updated` | Football Data Engine | `match_id, home_score, away_score, minute, period` | WebSocket Server, Dynamic Layout Purger |
| `match.concluded` | Football Data Engine | `match_id, final_score, stats` | Video Intelligence, Post-Match Summarizer |
| `content.auto_published`| Decision Engine | `published_id, section_code, slug` | Edge CDN Purge Worker |