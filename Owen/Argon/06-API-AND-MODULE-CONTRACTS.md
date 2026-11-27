# 06 — API & MODULE CONTRACTS

**Document Ref:** TC-DOC-06-CONTRACTS  
**System:** Internal REST Interfaces, BullMQ Job Schemas & Dynamic Layout Contracts  
**Architectural Authority:** Level 2 Interface Specification  

---

## 1. Dynamic Homepage REST API Contract

### `GET /api/v1/layout/homepage`
- **Description:** Supplies the full dynamic layout composition to The Champion frontend.
- **Cache-Control:** `public, s-maxage=15, stale-while-revalidate=45`
- **Response Schema:**

```json
{
  "status": "success",
  "data": {
    "state": "LIVE_MATCH",
    "club_context": {
      "club_id": "club_manchester_united",
      "club_name": "Manchester United",
      "active_match_id": "9a12c8b0-8f12-4211-b841-f76192837411"
    },
    "sections": [
      {
        "id": "sec_hero_live",
        "type": "MATCH_CENTER",
        "priority": 1,
        "title": "Pusat Pertandingan Langsung",
        "refresh_interval_seconds": 15,
        "payload": {
          "match_id": "9a12c8b0-8f12-4211-b841-f76192837411",
          "minute": 68,
          "period": "2H",
          "home_team": { "name": "Manchester United", "tla": "MUN", "score": 2 },
          "away_team": { "name": "Arsenal", "tla": "ARS", "score": 1 },
          "live_timeline": [
            { "minute": 24, "type": "GOAL", "player": "Bruno Fernandes", "team": "MUN" },
            { "minute": 52, "type": "GOAL", "player": "Bukayo Saka", "team": "ARS" },
            { "minute": 63, "type": "GOAL", "player": "Rasmus Højlund", "team": "MUN" }
          ]
        }
      },
      {
        "id": "sec_transfer_rumour",
        "type": "TRANSFER_HUB",
        "priority": 2,
        "title": "Bursa Transfer",
        "refresh_interval_seconds": 300,
        "payload": {
          "items": [
            {
              "id": "c1092834-4123",
              "title": "United Sepakati Kesepakatan Personal dengan Gelandang Muda",
              "badge": "RUMOUR",
              "confidence_score": 0.82,
              "source_name": "Fabrizio Romano",
              "source_url": "https://twitter.com/example",
              "published_at": "2026-11-27T06:30:00Z"
            }
          ]
        }
      }
    ],
    "meta": {
      "generated_at": "2026-11-27T08:00:00Z",
      "edge_cache_ttl": 15
    }
  }
}
```

---

## 2. Live Match Status Endpoint

### `GET /api/v1/matches/live`
- **Description:** Real-time polling endpoint for frontend SWR clients during active matches.
- **Cache-Control:** `no-cache, private`
- **Response Schema:**

```json
{
  "status": "success",
  "data": {
    "match_id": "9a12c8b0-8f12-4211-b841-f76192837411",
    "status": "IN_PLAY",
    "minute": 68,
    "scores": { "home": 2, "away": 1 },
    "stats": {
      "possession": { "home": 48, "away": 52 },
      "shots_on_target": { "home": 6, "away": 4 },
      "xg": { "home": 1.84, "away": 1.21 }
    }
  }
}
```

---

## 3. BullMQ Job Queue Event Payloads

### A. Queue: `queue:sports:critical`
```typescript
interface SportsSyncJob {
  jobType: 'POLL_LIVE_EVENTS' | 'SYNC_FIXTURES' | 'UPDATE_STANDINGS';
  matchId?: string;
  provider: 'SPORTMONKS' | 'API_FOOTBALL';
  priority: 1; // Highest priority
}
```

### B. Queue: `queue:content:background`
```typescript
interface ContentProcessJob {
  jobType: 'EXTRACT_AND_ENRICH';
  rawItemId: string; // UUID in raw_items
  sourceId: string;
  sourceTier: 1 | 2 | 3;
}
```

### C. Queue: `queue:video:sync`
```typescript
interface VideoSyncJob {
  jobType: 'SYNC_PLAYLIST';
  channelId: string;
  playlistId: string;
  targetMatchId?: string;
}
```