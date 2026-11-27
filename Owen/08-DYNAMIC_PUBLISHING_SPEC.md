# 08 — DYNAMIC PUBLISHING SPECIFICATION

**Project:** The Champion — It's All About MU  
**Subsystem:** 4-State Homepage Composition Engine & Dynamic JSON Contracts  
**Document Ref:** TC-DOC-08-DYN-PUB  
**Authoritative Hierarchy:** Level 2 Specification  

---

## 1. Four Dynamic Homepage States

The presentation layer of The Champion adapts dynamically based on Manchester United's live match lifecycle. The homepage does not present a static chronological blog feed; it organizes sections based on fan intent.

```
       [State 1: PRE-MATCH]                   [State 2: LIVE-MATCH]
  (Kickoff - 48h to Kickoff)               (Kickoff to FT + 15m)
+-------------------------------+      +-------------------------------+
| Hero: Match Preview & Countdown|     | Hero: Live Match Center (xG)  |
+-------------------------------+      +-------------------------------+
| Team News & Starting Lineup   |      | In-Game Stats & Events Stream |
+-------------------------------+      +-------------------------------+
| Head-to-Head & Form Guide     |      | Live Premier League Table     |
+-------------------------------+      +-------------------------------+

       [State 3: POST-MATCH]                  [State 4: NO-MATCH]
     (FT to FT + 24 Hours)                    (Midweek / Routine)
+-------------------------------+      +-------------------------------+
| Hero: Match Result & Highlights|     | Hero: Breaking News / Transfer|
+-------------------------------+      +-------------------------------+
| Tactical Report & Quotes      |      | Transfer Hub (Confirmed/Rumour|
+-------------------------------+      +-------------------------------+
| Watch Again Replays           |      | Upcoming Fixture Teaser       |
+-------------------------------+      +-------------------------------+
| Updated League Table Impact   |      | Player Spotlights & Archives  |
+-------------------------------+      +-------------------------------+
```

---

## 2. State Composition & Priority Matrix

| State Name | Activation Condition | Section Composition (Ordered Priority) | Cache TTL |
| :--- | :--- | :--- | :--- |
| **PRE_MATCH** | Next match kickoff $\le 48$ hours away | 1. `HERO_PREVIEW` (Countdown, Venue, Broadcaster)<br/>2. `CONFIRMED_LINEUPS` (T-60m)<br/>3. `HEAD_TO_HEAD`<br/>4. `RECENT_FORM_STANDINGS` | 60 sec (T-60m: 15s) |
| **LIVE_MATCH** | Current time between Kickoff and FT + 15 min | 1. `HERO_LIVE_CENTER` (Clock, Score, Scorecard)<br/>2. `LIVE_TIMELINE` (VAR, Goals, Subs)<br/>3. `IN_GAME_STATS` (xG, Possession)<br/>4. `LIVE_TABLE_TRACKER` | 15 sec |
| **POST_MATCH** | FT whistle confirmed up to FT + 24 hours | 1. `HERO_RESULT_REPLAY` (Score, Official Video Embed)<br/>2. `TACTICAL_SUMMARY`<br/>3. `POST_MATCH_REACTIONS`<br/>4. `UPDATED_STANDINGS` | 60 sec |
| **NO_MATCH** | No fixture scheduled within 48 hours | 1. `HERO_EDITORIAL_NEWS`<br/>2. `TRANSFER_INTELLIGENCE_GRID`<br/>3. `NEXT_FIXTURE_TEASER`<br/>4. `LEAGUE_STANDINGS`<br/>5. `HISTORIC_VAULT` | 300 sec |

---

## 3. Dynamic Section Layout JSON Contract

The frontend Next.js application renders the entire homepage dynamically by requesting `/api/v1/layout/homepage`.

### JSON Contract Payload Example:
```json
{
  "status": "success",
  "data": {
    "state": "POST_MATCH",
    "club_context": {
      "club_id": "club_manchester_united",
      "club_name": "Manchester United",
      "active_match_id": "9a12c8b0-8f12-4211-b841-f76192837411"
    },
    "sections": [
      {
        "id": "sec_hero_result",
        "type": "HERO_RESULT_REPLAY",
        "priority": 1,
        "title": "Hasil Pertandingan & Cuplikan Resmi",
        "refresh_interval_seconds": 60,
        "payload": {
          "match_id": "9a12c8b0-8f12-4211-b841-f76192837411",
          "home_team": { "name": "Manchester United", "score": 2, "tla": "MUN" },
          "away_team": { "name": "Liverpool", "score": 1, "tla": "LIV" },
          "status": "FINISHED",
          "period": "FT",
          "competition": "Premier League",
          "video_highlight": {
            "youtube_id": "dQw4w9WgXcQ",
            "title": "HIGHLIGHTS: Man Utd 2-1 Liverpool",
            "is_embeddable": true
          }
        }
      },
      {
        "id": "sec_tactical_summary",
        "type": "EDITORIAL_ARTICLE",
        "priority": 2,
        "title": "Analisis Pertandingan",
        "refresh_interval_seconds": 300,
        "payload": {
          "title": "Disiplin Taktis Kunci Kemenangan United di Old Trafford",
          "summary_bullets": [
            "Transisi serangan balik cepat membuahkan dua gol babak pertama",
            "Lini pertahanan kokoh menahan gempuran 15 menit terakhir",
            "Tambahan tiga poin krusial mendongkrak posisi di klasemen"
          ],
          "provenance_source": "The Athletic",
          "provenance_url": "https://theathletic.com/example"
        }
      }
    ],
    "meta": {
      "generated_at": "2026-11-27T07:45:00Z",
      "edge_cache_ttl": 60
    }
  }
}
```