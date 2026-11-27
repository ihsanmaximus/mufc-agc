# 06 — CONTENT INTELLIGENCE SPECIFICATION

**Project:** The Champion — It's All About MU  
**Subsystem:** Text Harvesting, Multi-Tier Deduplication & Transformative Indonesian Synthesis  
**Document Ref:** TC-DOC-06-CONTENT-INTEL  
**Authoritative Hierarchy:** Level 2 Specification  

---

## 1. Whitelisted Source Hierarchy & Policy

To guarantee journalistic integrity and mitigate copyright exposure, content harvesting is strictly restricted to whitelisted sources grouped into three confidence tiers:

- **Tier 1 (Authoritative / Official):**
  - Official Manchester United Media (ManUtd.com, Club Press Office).
  - Verified Broadcasters: BBC Sport, Sky Sports Football.
  - Tier-1 Football Journalists: David Ornstein (The Athletic), Fabrizio Romano.
  - *Publishing Policy:* Automatically eligible for auto-publishing if $S \ge 0.88$.
- **Tier 2 (Reliable Editorial):**
  - The Guardian Football, The Telegraph, ESPN UK, Manchester Evening News (selected club correspondents).
  - *Publishing Policy:* News eligible for auto-publish; transfer rumours held for human moderation.
- **Tier 3 (Speculative / Tabloid Feeds):**
  - Daily Mail, The Sun, various fan aggregate blogs.
  - *Publishing Policy:* Strictly quarantined or held for manual review. Never auto-published.

---

## 2. Multi-Tier Deduplication Architecture

Before any article is submitted for LLM translation or published to the frontend, it passes through three sequential deduplication barriers:

```
[Incoming Extracted Item]
          │
          ├──> (1) Exact MD5 Hash Match (URL & Raw Text) ──> [Match Found: DISCARD]
          │
          ├──> (2) Syntactic MinHash Fingerprint (> 85%) ──> [Match Found: MERGE SOURCE ATTRIBUTION]
          │
          └──> (3) Semantic pgvector Cosine (> 0.92)    ──> [Match Found: ATTACH TO STORY CLUSTER]
```

1. **Exact MD5 Check:** Computes MD5 on canonicalized raw text. Duplicate URLs or exact syndicate reprints are discarded immediately.
2. **MinHash / SimHash Syntactic Check:** Strips punctuation and stop words; compares 64-bit token shingles. Detects syndication where minor word modifications exist.
3. **Semantic Vector Search:** Computes 768-dimensional text embeddings. If cosine similarity $> 0.92$ against items within the last 36 hours, it attaches the new report as a source reference to the existing `story_clusters` record, avoiding redundant homepage cards.

---

## 3. Indonesian Synthesis & Prompt Engineering

The LLM is utilized strictly for **transformative synthesis** to adhere to fair-use standards and provide immediate fan utility.

### Structured JSON System Prompt Specification:
```json
{
  "system_instruction": "You are the Senior Indonesian Sports Editor for 'The Champion'. Transform the following English football report into a structured Indonesian editorial brief. Maintain an objective, journalistic, fan-first tone. Never invent statistics, dates, or match outcomes not present in the source.",
  "temperature": 0.2,
  "response_format": {
    "type": "json_schema",
    "schema": {
      "type": "object",
      "properties": {
        "title_id": { "type": "string", "description": "Punchy Indonesian editorial title under 80 characters" },
        "summary_bullets": {
          "type": "array",
          "items": { "type": "string" },
          "minItems": 3,
          "maxItems": 3,
          "description": "Three key takeaways in Indonesian"
        },
        "editorial_body_id": { "type": "string", "description": "Concise Indonesian editorial narrative (150-200 words)" },
        "transfer_category": { "type": "string", "enum": ["CONFIRMED", "ADVANCED", "RUMOUR", "NONE"] },
        "named_entities": {
          "type": "object",
          "properties": {
            "players_mentioned": { "type": "array", "items": { "type": "string" } },
            "manager_mentioned": { "type": "boolean" },
            "opponent_club": { "type": "string" }
          },
          "required": ["players_mentioned", "manager_mentioned"]
        },
        "extracted_quotes": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "speaker": { "type": "string" },
              "quote_original": { "type": "string" },
              "quote_id": { "type": "string" }
            }
          }
        }
      },
      "required": ["title_id", "summary_bullets", "editorial_body_id", "transfer_category", "named_entities"]
    }
  }
}
```

---

## 4. Provenance & Copyright Compliance

- **Fair-Use Mandate:** The original full-text body of third-party articles is **never** stored or rendered to public users.
- **Mandatory Attribution Card:** Every published editorial brief displays:
  - Canonical source logo and publisher name.
  - Original publication timestamp.
  - Clear hyperlink: *"Baca artikel sumber asli di [Publisher Name] ->"*.