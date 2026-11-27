# 08 — FRONTEND FAN EXPERIENCE & INTERFACE ARCHITECTURE

**Document Ref:** TC-DOC-08-FRONTEND  
**System:** The Champion Supporter Experience & Client Architecture  
**Architectural Authority:** Level 2 Client Specification  

---

## 1. Fan-First UX Philosophy

Traditional football news sites present a wall of text articles. The Champion prioritizes supporter intent:
1. **Matchday Awareness:** If United is playing right now, the live match clock, score, and in-game events dominate the screen above the fold.
2. **Instant Cognitive Relief:** Editorial items open with a 3-bullet core takeaway in bold before presenting the narrative.
3. **Verified Highlights:** Fans can watch official YouTube replays directly in the portal without navigating to external apps.
4. **Zero Layout Shifts:** High-performance skeleton loaders ensure stability as dynamic sections hydrate.

---

## 2. Core Section Catalog

| Section Identifier | Visual Purpose | Primary Target Data Source |
| :--- | :--- | :--- |
| `HERO_MATCH_LIVE` | Real-time score, minute clock, timeline of goals/cards | Authoritative Sports API |
| `HERO_PREVIEW` | Upcoming fixture countdown, opponent crest, venue, broadcaster | Authoritative Sports API |
| `HERO_RESULT` | Final scoreline, scorers, match report, highlight embed | Sports API + YouTube API |
| `HERO_EDITORIAL` | High-confidence breaking news feature digest | Content Intelligence Engine |
| `WATCH_AGAIN` | Grid of official club and broadcaster highlight embeds | Video Intelligence Engine |
| `TRANSFER_HUB` | Filterable transfer intelligence grid (Confirmed vs. Rumour) | Content Intelligence Engine |
| `STANDINGS_WIDGET`| Compact Premier League table highlighting MU's live rank | Authoritative Sports API |
| `NEWS_FEED` | Secondary chronological digest of club updates | Content Intelligence Engine |

---

## 3. Responsive Breakpoints & Device Adaptation

```
+-----------------------------------------------------------------------------------+
| BREAKPOINT      | VIEWPORT RANGE | PRIMARY LAYOUT BEHAVIOR                        |
+-----------------------------------------------------------------------------------+
| Mobile (Base)   | 360px - 767px  | Single column; sticky live scorebar on scroll; |
|                 |                | horizontal swipeable cards for lineups.        |
| Tablet          | 768px - 1023px | 2-column grid; match timeline side-by-side    |
|                 |                | with in-game stats widget.                    |
| Desktop         | 1024px - 1440px| 3-column structured portal; persistent table   |
|                 |                | sidebar; high-resolution video player embed.   |
+-----------------------------------------------------------------------------------+
```